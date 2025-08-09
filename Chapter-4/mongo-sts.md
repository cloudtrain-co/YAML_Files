```bash
oc new-project mongo
oc apply -f Chapter-4/mongo-sts-with-probes.yaml
```
#Update the liveness probe
```bash
oc patch sts/mongo -n mongo --type='json' -p="[{'op':'replace','path':'/spec/template/spec/containers/0/livenessProbe/exec/command/2','value':'db.adminxCommand(\"ping\")'}]"
```
```bash
oc rollout restart sts/mongo -n mongo
oc rollout status sts/mongo -n mongo
```
```bash
oc describe pod mongo-1 -n mongo
```
#Readiness Probe
```bash
oc patch statefulset mongo -n mongo --type='json' -p='[{"op":"replace","path":"/spec/template/spec/containers/0/readinessProbe/tcpSocket/port","value":27018}]'
```
```bash
oc rollout restart sts/mongo -n mongo
oc rollout status sts/mongo -n mongo
```
```bash
oc describe pod mongo-1 -n mongo
```
