### Working with labels and selectors
```bash
oc apply -f pod.yaml
oc get po
oc get po/my-pod --show-labels
oc label pod/my-pod release=dev
oc label pod/my-pod release=devenv –overwrite=true #Overwrite the existing label
oc label pod/my-pod release- #Remove the label
oc get po -l release=dev
oc label pod/my-pod env=dev
oc label pod/my-pod env=prod
oc get po -l 'env in (dev, prod)' #Show pods with given matching lables 
```
