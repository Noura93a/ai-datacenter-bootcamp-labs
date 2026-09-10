# Lab W4D2: make it self-healing

Start:      Yesterday's kind cluster and your image loaded into it
            (`kind get clusters` shows `aidc`; delete yesterday's bare pod if
            it is still there: `kubectl delete pod serving --ignore-not-found`).
Objective:  Replace the bare pod with a Deployment behind a Service, gate
            traffic with probes, then prove the two claims that justify all of
            it: a killed pod comes back by itself, and a rolling update drops
            zero requests while a client is watching.

Time: about 3 hours. Tier 0 throughout (kind, echo backend).

Yesterday you were the restart policy. Today you resign: the Deployment holds
the desired state, the Service holds the stable name, and the probes hold the
line on what "ready" means. The afternoon ends with the kill demo from the
morning's lecture reproduced by you, plus the number that matters in
production: dropped requests through an update, and it had better be zero.

## Predict (by hand)

Credited for handing it in, never marked right or wrong - hedged guesses teach nothing, and nothing here is graded for accuracy.

On the card before you apply anything.

- You delete one of the two pods while a client inside the cluster hits the
  Service every 100 ms. How many requests fail: zero, a handful, or half of
  them until the pod returns?
- A rolling update replaces both pods with readiness probes gating traffic.
  Same client. How many failures this time? Commit to a number.
- The app answers 503 on `/health` until its backend loads. Which probe is
  that for, readiness or liveness, and what would go wrong if you used it for
  the other one?
- A roll under your constraint card's settings (Step 6): how many failures,
  and does your card accept them?

## The delta

### Step 1: Deployment and Service, given skeletons (about 25 min)

`deployment.yaml` here is yesterday's pod inside a Deployment: 2 replicas, a
RollingUpdate strategy with `maxUnavailable: 0`, and a `preStop` sleep whose
comment you should read now, because Step 5 measures what it is for. Edit the
image line to yours, then:

```bash
kubectl apply -f deployment.yaml -f service.yaml
kubectl rollout status deployment/serving
kubectl get pods -l app=serving
```

Two pods, names generated from the Deployment's hash. Read `service.yaml`
while they start: the Service selects the label `app: serving`, not the pod
names, which is why it keeps working while pods come and go.

### Step 2: the probes (about 30 min)

The two TODO blocks in `deployment.yaml`. Both point at `GET /health:8000`,
and they answer different questions:

- **readiness**: "may traffic come here?" The app's honest 503-until-loaded
  makes this real: a pod that is `Running` but not ready receives nothing.
  `periodSeconds: 2`, `failureThreshold: 2` is a fine starting answer.
- **liveness**: "should this pod be restarted?" Generous
  `initialDelaySeconds` (20 is fine at echo speeds), because a liveness probe
  that fires during a slow model load kills a pod that was about to be fine,
  in a loop, forever. That exact loop is Thursday's most popular failure mode
  on real clusters.

```bash
kubectl apply -f deployment.yaml
kubectl rollout status deployment/serving
kubectl describe deployment serving | grep -A2 -E 'Liveness|Readiness'
```

### Step 3: an in-cluster client (about 20 min)

Port-forward pins itself to one pod, so it is the wrong instrument for
watching a Service balance across pods. Run the watcher inside the cluster,
using your own image (it carries Python; nothing new to pull):

```bash
kubectl run prober --image=<your-user>/aidc-serving:cpu-v1 --restart=Never --command -- \
  python -c '
import time, urllib.request
ok = bad = 0
end = time.time() + 90
while time.time() < end:
    try:
        with urllib.request.urlopen("http://serving:8000/health", timeout=2) as r:
            ok += (r.status == 200)
    except Exception:
        bad += 1
    time.sleep(0.1)
print(f"PROBE RESULT ok={ok} bad={bad}")
'
```

`http://serving:8000` is the Service by DNS name. This pod runs 90 seconds and
prints one line; everything in Steps 4 and 5 happens while one of these is
running.

### Step 4: the kill demo, yours now (about 20 min)

Start a prober, then in another terminal:

```bash
kubectl get pods -l app=serving          # pick one
kubectl delete pod <one-of-them>
kubectl get pods -l app=serving -w       # watch the replacement arrive
```

Then read the prober's verdict:

```bash
kubectl logs prober; kubectl delete pod prober
```

Expect `bad=0`: the Service routed around the dying pod, the Deployment
replaced it, and no client noticed. That is the morning's kill demo, measured
instead of watched.

### Step 5: the rolling update, measured (about 30 min)

Start a fresh prober, then trigger an update while it watches:

```bash
kubectl set env deployment/serving APP_VERSION=v2
kubectl rollout status deployment/serving
kubectl logs prober          # after it finishes; then delete it
```

The bar is `bad=0`, and it is earned by three settings working together:
`maxUnavailable: 0` (never remove capacity first), the readiness probe (no
traffic to a pod that cannot serve), and the `preStop` sleep. That last one is
the subtle one: without it, a terminating pod stops accepting the instant it
is told to die, but the cluster takes a moment to remove it from the Service's
endpoints, and requests landing in that gap fail. Building this lab, the same
run measured **2 failures in a 45-second run without `preStop`, 0 with it**. Delete
the `lifecycle:` block, re-run this step, and you can reproduce the 2; put it
back and the 0 returns.

### Step 6: your constraint card's roll (about 20 min)

The morning gave your team a constraint card: A (availability first), B (speed
first) or C (no headroom). Now price your own card. Edit `strategy:` to the
settings your team chose for it, predict the failure count on your prediction
card, and roll once under a fresh prober:

```bash
kubectl set env deployment/serving APP_VERSION=v3
kubectl rollout status deployment/serving
kubectl logs prober
```

Record three things on the card: the settings, the measured `bad=` count, and
one sentence - would the card's owner sign off on that number? A card-C roll
(`maxUnavailable: 1, maxSurge: 0`) with 2 replicas usually drops requests
during the dip; that is not a bug, it is the cost your card accepted, now
measured. A card-B roll (`maxSurge: 2`) finishes faster; watch
`rollout status` and note by how much.

Then restore `{maxUnavailable: 0, maxSurge: 1}` and the `lifecycle:` block if
you touched it: the green check demands the zero bar, whatever your card said.
The card is judgment recorded; the gate is the floor everyone ships on.

### Step 7: green check (about 10 min)

```bash
bash verify.sh
```

## Verify (green check)

`verify.sh` re-runs the whole claim, not your memory of it: it checks the
Deployment has 2 ready replicas, both probes and the preStop in its live spec,
then starts its own in-cluster prober, triggers a rolling update by flipping
`APP_VERSION`, waits for both to finish, and demands the prober counted zero
failures. Expected final line: `GREEN CHECK: PASS`.

## Stretch

Scale to `replicas: 3` with the prober running (`kubectl scale deployment
serving --replicas=3`), then back to 2. Then run another team's constraint
card the way you ran yours in Step 6: a card you did not argue for is the best
test of whether the settings follow from the constraint or from habit.

## Failure modes

- **Both pods `0/1 READY` after Step 2.** The readiness probe is pointing at
  the wrong port or path, so nothing ever becomes ready and the rollout
  wedges. `kubectl describe pod <one>` shows the probe and its last failure.
- **Pods restart in a loop with `CrashLoopBackOff` after adding probes.** The
  liveness probe is too impatient for startup. Raise `initialDelaySeconds`;
  readiness tolerance is cheap, liveness impatience is fatal.
- **Prober prints `bad` in the hundreds during an update.** The readiness
  probe is missing (traffic reaches unready pods) or `maxUnavailable` was left
  at default (capacity removed before replacement). Check both against Step 1's
  skeleton.
- **A handful of failures (1-5) during the update, everything else right.**
  That is the preStop gap. Confirm the `lifecycle:` block survived your edits
  and that your cluster is 1.30+ (`kubectl version`) for the native sleep
  action.
- **`kubectl run prober` fails with ImagePullBackOff.** The prober uses your
  image so nothing new needs pulling, but it still needs the exact tag you
  `kind load`ed yesterday; match it character for character.
