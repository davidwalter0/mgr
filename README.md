```
sed  -i -e 's,# setsebool -P docker_transition_unconfined 1,# setsebool -P docker_transition_unconfined 1,g' -e "s,OPTIONS='--selinux-enabled',# OPTIONS='--selinux-enabled',g" /etc/sysconfig/docker
```


```
#!/bin/bash
```

# Prototype the monitor / collaboration mgr component

0. each pod required for the test is already starting.

1. assume that each repo has a prepare.sh, run.sh, and cleanup.sh in
the root of the repo script as a common entry point.

prepare executes git pull repos then make each docker build or
whatever with.

POD DEF
   volumes:
```
TODO : insert mgr
```
2. Each member container in a POD has an environment variable
START_STEP=${WRK_MGR_DIR}/name [in some future versions maybe a file injected
into the pod as a volume or other files]

```
kubectl --kubeconfig=${KUBECONFIG} create -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: k8s-git-repo-test
  namespace: default
  labels:
    app: k8s-git-repo-test
spec:
  restartPolicy: Never
  containers:
  - name: k8s-busybox-pull-data
    privileged: true
    image: busybox
    imagePullPolicy: IfNotPresent
    args:
    - /bin/sh
    -  -c
    - "while ! [[ -e ${FILE} ]]; do sleep 1; done; /bin/cat ${FILE}"
    volumeMounts:
    - mountPath: /repo
      name: repos
      readOnly: false
    - mountPath: /mgr
      name: step
    env:
    - name: FILE
      value: /repo/README.md
    - name: WRK_START_STEP
      value: /mgr/step-00

  - name: mgr
    privileged: true
    image: ubuntu-15.04:dind
    imagePullPolicy: IfNotPresent
    args:
    - /mgr/mgr.sh
    env:
    - name: FILE
      value: /repo/README.md
    - name: WRK_START_STEP
      value: /mgr/step-00
    - name: WRK_MGR_DIR
      value: /mgr/step-00
    - name: GIT_REPO
      value: http://github.com/davidwalter0/repo.git
    volumeMounts:
    - mountPath: /mgr
      name: step
    - mountPath: /status
      name: status
    - mountPath: /repo
      name: repos
  volumes:
  - name: repos
    emptyDir: {}
  - name: secrets
    emptyDir: {}
  - name: mgr
    emptyDir: {}
  - name: steps
    emptyDir: {}
  - name: status
    emptyDir: {}
  #  hostPath:
  #     path: /mgr
```

yaml or json?

```
---

{
  "kind": "Secret",
  "apiVersion": "v1",
  "metadata": {
    "name": "ssh-key-secret"
  },
  "data": {
    "id-rsa": "dmFsdWUtMg0KDQo=",
    "id-rsa.pub": "dmFsdWUtMQ0K"
  }
}

```

```
---
{
  "kind": "Pod",
  "apiVersion": "v1",
  "metadata": {
    "name": "secret-test-pod",
    "labels": {
      "name": "secret-test"
    }
  },
  "spec": {
    "volumes": [
      {
        "name": "secret-volume",
        "secret": {
          "secretName": "ssh-key-secret"
        }
      }
    ],
    "containers": [
      {
        "name": "ssh-test-container",
        "image": "mySshImage",
        "volumeMounts": [
          {
            "name": "secret-volume",
            "readOnly": true,
            "mountPath": "/etc/secret-volume"
          }
        ]
      }
    ]
  }
}
```

```
---
apiVersion: v1
data:
  id-rsa: dmFsdWUtMg0KDQo=
  id-rsa.pub: dmFsdWUtMQ0K
kind: Secret
metadata:
  name: ssh-key-secret
```

```
---
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: secret-test
  name: secret-test-pod
spec:
  containers:
  - image: mySshImage
    name: ssh-test-container
    volumeMounts:
    - mountPath: /etc/secret-volume
      name: secret-volume
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: ssh-key-secret

```


```
for ((i=0; i<1; i++)); do
  for i in 0 1; do 
    bin/kubectl --kubeconfig=${KUBECONFIG} logs mgr01 step-0${i}; 
  done; 
  bin/kubectl --kubeconfig=${KUBECONFIG} logs  mgr01 mgr; sleep 1; 
done
```
