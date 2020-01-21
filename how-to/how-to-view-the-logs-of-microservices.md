# How to view the logs of microservices

## skycli app get-k8s-credentials

You can use kubectl to view logs as usual. To do this, you typically need to run `kubectl config set-cluster`, `kubectl config set-user` and `kubectl config set-context`. skycli has command to abstract all these details.

The version of kubectl must be &gt;= 1.15

```bash
$ skycli --app yourapp app get-k8s-credentials
# The necessary configuration was written to KUBECONFIG.
# Follow the instruction in the output to
# switch kubectl context if you wish.
```

Then you can verify the configuration was correctly set by skycli

```bash
$ kubectl config view
```

First you need to know the name of the pods

```bash
$ kubectl get pod
```

Finally you can view the logs with

```bash
$ kubectl logs mypod
```



