# Simple steps for setting up Distributed TensorFlow on Kubernetes using ACS-Engine

## 1. Create a Kubernetes cluster
Follow the instructions [here](https://github.com/Azure/acs-engine/blob/master/docs/kubernetes.md) to create a Kubernetes cluster using acs-engine.  
Note that you can use [Azure Container Service](https://docs.microsoft.com/en-us/azure/container-service/kubernetes/container-service-kubernetes-walkthrough) to create a Kubernetes cluster as well, but currently ACS does not support heterogeneous agent pools yet. So if you need a cluster with different size VMs, some with GPUs, you'll need to use acs-engine for now.

## 2. Setup GPU drivers on the agent host VMs with GPUs
Execute the scripts [here](https://github.com/Microsoft/DLWorkspace/blob/master/src/ClusterBootstrap/scripts/prepare_acs.sh) on each agent host VM with GPUs.

## 3. Create a set of pods for distributed TensorFlow
For example, to create 2 parameter servers and 2 worker, use these [sample yaml files](https://github.com/joyq-github/TensorFlowonK8s/tree/master/sampleyaml) to create pods and services in your cluster. Once done, run *kubectl get pods*, and you should see 4 pods returned. 

Run *kubectl get svc* to get the service IP addresses of each ps and worker node, as shown below.  
<pre>    tf-ps0       10.0.211.196   <none>        6006/TCP,2222/TCP      
    tf-ps1       10.0.81.168    <none>        6006/TCP,2222/TCP  
    tf-worker0   10.0.221.23    <none>        6006/TCP,2222/TCP  
    tf-worker1   10.0.118.248   <none>        6006/TCP,2222/TCP        
</pre>
## 4. Run distributed Tensorflow
For example, to run a distributed TensorFlow training job using 1 parameter server and 2 workers, execute the following commands in the order listed below.   
The sample tensorflow script used below can be downloaded from [here](https://github.com/tensorflow/benchmarks/blob/master/scripts/tf_cnn_benchmarks/tf_cnn_benchmarks.py).

a) on the parameter server pod, execute the script below:  
*python tf_cnn_benchmarks.py --local_parameter_device=cpu --num_gpus=4 \
--batch_size=128 --model=googlenet --variable_update=parameter_server --num_batches=200 --cross_replica_sync=False \
--data_name=imagenet --data_dir=/imagenetdata --job_name=ps --ps_hosts=10.0.211.196:2222 \
--worker_hosts=10.0.221.23:2222,10.0.118.248:2222 --task_index=0*

b) on the worker0 pod, execute the script below:  
*python tf_cnn_benchmarks.py --local_parameter_device=cpu --num_gpus=4 \
--batch_size=128 --model=googlenet --variable_update=parameter_server --num_batches=200 --cross_replica_sync=False \
--data_name=imagenet --data_dir=/imagenetdata --job_name=worker --ps_hosts=10.0.211.196:2222 \
--worker_hosts=10.0.221.23:2222,10.0.118.248:2222 --task_index=0*


c) on the worker1 pod, execute the script below:  
*python tf_cnn_benchmarks.py --local_parameter_device=cpu --num_gpus=4 \
--batch_size=128 --model=googlenet --variable_update=parameter_server --num_batches=200 --cross_replica_sync=False \
--data_name=imagenet --data_dir=/imagenetdata --job_name=worker --ps_hosts=10.0.211.196:2222 \
--worker_hosts=10.0.221.23:2222,10.0.118.248:2222 --task_index=1*
