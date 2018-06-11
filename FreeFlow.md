# Deploy FreeFlow plugin in Kubernetes 
(Credits: [Yibo Zhu](https://www.microsoft.com/en-us/research/people/yibzh/) from Microsoft Research)

Sometimes the pod-to-pod network bandwidth might not be as good as the host-to-host network bandwidth in your Kuberbetes cluster due to various reasons. And network bandwidth is often one of the most important factors for distributed model training performance. To optimize your pod-to-pod network bandwidth, the FreeFlow plugin (https://github.com/Microsoft/Freeflow) is very helpful and it’s easy to use with just 2 simple steps. 

## 1. Deploy FreeFlow as a DaemonSet in your Kubernetes cluster
See sample yaml [here](https://github.com/joyq-github/TensorFlowonK8s/blob/master/sampleyaml/freeflow.yaml).  In the yaml file, change the environment variable value for HOST_IP_PREFIX to the actual IP ranges that your pods use.

## 2. Create your pod with FreeFlow enabled. 
See sample yaml [here](https://github.com/joyq-github/TensorFlowonK8s/blob/master/sampleyaml/tfworkerwithfreeflow.yaml). Add 2 environment variables LD_PRELOAD and VNET_PREFIX into your pod definition, as shown below. Again, change the environment variable value for VNET_PREFIX to the actual IP ranges that your pods use.
<pre>
      containers:
      - name: tf-worker1
        image: tensorflow/tensorflow:1.8.0-gpu
        env:
        - name: LD_PRELOAD
          value: "/freeflow/libfsocket.so"
        - name: VNET_PREFIX
          value: 10.244.0.0/16
</pre>
And also mount the volume /freeflow that contains the FreeFlow library in the pod. 
<pre>
        volumeMounts:
        - mountPath: /freeflow
          name: freeflow      
      volumes:
      - name: freeflow
        hostPath:
          path: /freeflow  
</pre>

That’s it! You should now have FreeFlow enabled for your pods for optimized network bandwidth performance. 


