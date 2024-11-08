# topic-user-standalone
 
## Process for Deployment

Following the documentation: https://docs.redhat.com/en/documentation/red_hat_streams_for_apache_kafka/2.7/html/deploying_and_managing_streams_for_apache_kafka_on_openshift/index


1.  Deploy Cluster Operator for "Streams for Apache Kafka" via Operator Hub or using manifest files under the cluster-operator/ directory.  They currently are configured for all namespaces, that is namespace parameters are set to "*".

2.  Deploy a Kafka Broker cluster using one of the following:

examples/kafka/kafka-ephemeral-single-simpleauth.yaml - has authorization set to simple as in:
```
   authorization:
      type: simple
      superUsers:
        - user1
```        
This also has the EntityOperator element removed.

examples/kafka/kafka-ephemeral-single-sinEntity.yaml -
This is identical to kafka-ephemeral-single.yaml but has the EntityOperator element removed.

**NOTE: This is deployed in POC currently - in the *kafka-ns* namespace**

examples/kafka/kafka-ephemeral-single.yaml -
The original example ephemeral Kafka broker and contains
```
  entityOperator:
    topicOperator: {}
    userOperator: {}
```
This creates the EntityOperator and 2 pods, one each for the Topic and User Operators.  However since we are interested in Standalone per namespace, these don't seem to be used.


3.  Deploy a Topic Operator in Namespaces Dept01, Dept02
Following the guidance in the documentation: https://docs.redhat.com/en/documentation/red_hat_streams_for_apache_kafka/2.7/html/deploying_and_managing_streams_for_apache_kafka_on_openshift/deploy-tasks_str#deploying-the-topic-operator-standalone-str

3.1  Create a namespace for dept01 and dept02
3.2  from the topic-operator-dept1/ directory run the following:
```
oc create -f 01-ServiceAccount-strimzi-topic-operator.yaml -n dept01
serviceaccount/strimzi-topic-operator created

oc create -f 02-Role-strimzi-topic-operator.yaml -n dept01
role.rbac.authorization.k8s.io/strimzi-topic-operator created

oc create -f 03-RoleBinding-strimzi-topic-operator.yaml -n dept01
rolebinding.rbac.authorization.k8s.io/strimzi-topic-operator created

oc create -f 05-Deployment-strimzi-topic-operator.yaml -n dept01
deployment.apps/strimzi-topic-operator created

```
Verify in dept01 namespace that the deployment of the strimzi-topic-operator is successful.


3.3  Repeat Process for dept02 Namespace


4.  Create User Topic Operator
Here is where the authentication failures start.  Trying the Kafka Broker simple authentication config seems to not resolve the problem. 

4.1  Deploy user-operator-dept1/ manifests
```
oc create -f 01-ServiceAccount-strimzi-user-operator.yaml -n dept01
serviceaccount/strimzi-user-operator created

oc create -f 02-Role-strimzi-user-operator.yaml  -n dept01
role.rbac.authorization.k8s.io/strimzi-user-operator created

oc create -f 03-RoleBinding-strimzi-user-operator.yaml           -n dept01
rolebinding.rbac.authorization.k8s.io/strimzi-user-operator created

oc create -f 05-Deployment-strimzi-user-operator.yaml            -n dept01
deployment.apps/strimzi-user-operator created
```

Verify that pod logs show no errors.  Latest errors are: 
```
2024-11-07 20:45:09 WARN  QuotasCache:65 - Failed to load Quotas
java.util.concurrent.ExecutionException: org.apache.kafka.common.errors.ClusterAuthorizationException: Cluster authorization failed.
```

Your mileage may vary ...