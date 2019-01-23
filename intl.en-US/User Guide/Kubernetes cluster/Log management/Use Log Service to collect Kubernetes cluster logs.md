# Use Log Service to collect Kubernetes cluster logs {#concept_nsj_sxm_t2b .concept}

Log Service is integrated with Kubernetes clusters of Alibaba Cloud Container Service. You can enable Log Service when creating a cluster to quickly collect container logs for the Kubernetes cluster, such as the standard output of the container and text files of the container.

## Enable Log Service when creating a Kubernetes cluster {#section_ohf_y5r_gfb .section}

If you have not created any Kubernetes clusters, follow steps in this section to enable Log Service:

1.  Log on to the [Container Service console](https://partners-intl.console.aliyun.com/#/cs).
2.  Click **Clusters** in the left-side navigation pane and click **Create Kubernetes Cluster** in the upper-right corner.
3.  For how to configure a cluster on the creation page, see [Create a Kubernetes cluster](../../../../../reseller.en-US/User Guide/Kubernetes cluster/Clusters/Create a cluster.md#).
4.  Drag to the bottom of the page and select the **Using Log Service** check box. The log plug-in will be installed in the newly created Kubernetes cluster.
5.  When you select the Using Log Service check box, project options are displayed. A project is the unit in Log Service to manage logs. For more information about projects, see [Project ](../../../../../reseller.en-US/Product Introduction/Basic concepts/Project .md#). Currently, two ways of using a project are available:
    -   Select an existing project to manage collected logs.

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/17400/15474591659250_en-US.png)

    -   The system automatically creates a new project to manage collected logs. The project is automatically named `k8s-log-{ClusterID}`, where ClusterID represents the unique identifier of the created Kubernetes cluster.

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/17400/15474591659251_en-US.png)

6.  After you complete the configurations, click **Create** in the upper-right corner. In the displayed dialog box, click **OK**.

    After the cluster creation is completed, the newly created Kubernetes cluster is displayed on the cluster list page.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/17400/15474591669449_en-US.png)


## Manually install Log Service components in a created Kubernetes cluster {#section_shf_y5r_gfb .section}

If you have created a Kubernetes cluster, following instructions in this section to use Log Service:

-   Log Service components are not installed. Manually install the components.
-   Log Service components are installed but in an earlier version. Upgrade the components. If you do not upgrade the components, you can only use the Log Service console or custom resource definition \(CRD\) to configure log collection.

**Check the Log Service component version**

1.  Configure the local kubeconfig to connect to the Kubernetes cluster through kubectl.

    For information about the configuration, see [Connect to a Kubernetes cluster by using kubectl](reseller.en-US/User Guide/Kubernetes cluster/Cluster management/Connect to a Kubernetes cluster by using kubectl.md#).

2.  Run the following command to fast determine whether an upgrade or migration operation is required:

    ```
    $ kubectl describe daemonsets -n kube-system logtail-ds | grep ALICLOUD_LOG_DOCKER_ENV_CONFIG
    
    ```

    -   If `ALICLOUD_LOG_DOCKER_ENV_CONFIG: true` is output, the components can be used directly without requiring upgrade or migration.
    -   If other results are output, check the components further.
3.  Run the following command to determine whether Helm is used to install the components.

    ```
    $ helm get alibaba-log-controller | grep CHART
    CHART: alibaba-cloud-log-0.1.1
    
    ```

    -   0.1.1 in the output indicates the version of the Log Service components. Please use the version of 0.1.1 and later. If the version is too early, please see [Upgrade Log Service components](#section_b3f_y5r_gfb) to upgrade the components. If you have used Helm to install the components of a valid version, you can skip next steps.
    -   If no results are output, the components are installed by using Helm. But the DaemonSet installation method might be used. Follow the next step to check further.
4.  DaemonSet can be an old one or a new one:

    ```
    $ kubectl get daemonsets -n kube-system logtail
    
    ```

    -   If no result is output or `No resources found.` is output, the Log Service components are not installed. For information about the installation method, see [Manually install Log Service components](#section_zhf_y5r_gfb).
    -   If the correct result is output, an old DaemonSet is used to install the components which require upgrade. For information about upgrading the components, see [Upgrade Log Service components](#section_b3f_y5r_gfb).

## Manually install Log Service components {#section_zhf_y5r_gfb .section}

1.  Configure the local kubeconfig to connect to the Kubernetes cluster through kubectl.

    For information about the configuration, see [Connect to a Kubernetes cluster by using kubectl](reseller.en-US/User Guide/Kubernetes cluster/Cluster management/Connect to a Kubernetes cluster by using kubectl.md#).

2.  Replace a parameter and run the following command.

    Replace $\{your\_k8s\_cluster\_id\} in the following command with your Kubernetes cluster ID, and then run the command.

    ```
    wget https://acs-logging.oss-cn-hangzhou.aliyuncs.com/alicloud-k8s-log-installer.sh -O alicloud-k8s-log-installer.sh; chmod 744 ./alicloud-k8s-log-installer.sh; ./alicloud-k8s-log-installer.sh --cluster-id ${your_k8s_cluster_id} --ali-uid ${your_ali_uid} --region-id ${your_k8s_cluster_region_id}
    ```

    **Parameter descriptions:**

    -   your\_k8s\_cluster\_id: You Kubernetes cluster ID.
    -   your\_ali\_uid: You account ID of Alibaba Cloud, which can be viewed in the user info.

**Installation example**

```
[root@iZbp******biaZ ~]# wget https://acs-logging.oss-cn-hangzhou.aliyuncs.com/alicloud-k8s-log-installer.sh -O alicloud-k8s-log-installer.sh; chmod 744 ./alicloud-k8s-log-installer.sh; ./alicloud-k8s-log-installer.sh --cluster-id c77a*****************0106 --ali-uid 19*********19 --region-id cn-hangzhou
--2018-09-28 15:25:33--  https://acs-logging.oss-cn-hangzhou.aliyuncs.com/alicloud-k8s-log-installer.sh
Resolving acs-logging.oss-cn-hangzhou.aliyuncs.com... 118.31.219.217, 118.31.219.206
Connecting to acs-logging.oss-cn-hangzhou.aliyuncs.com|118.31.219.217|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2273 (2.2K) [text/x-sh]
Saving to: ‘alicloud-k8s-log-installer.sh’

alicloud-k8s-log-installer.sh                      100%[================================================================================================================>]   2.22K  --. -KB/s    in 0s

2018-09-28 15:25:33 (13.5 MB/s) - ‘alicloud-k8s-log-installer.sh’ saved [2273/2273]

--2018-09-28 15:25:33--  http://logtail-release-cn-hangzhou.oss-cn-hangzhou.aliyuncs.com/kubernetes/alibaba-cloud-log.tgz
Resolving logtail-release-cn-hangzhou.oss-cn-hangzhou.aliyuncs.com... 118.31.219.49
Connecting to logtail-release-cn-hangzhou.oss-cn-hangzhou.aliyuncs.com|118.31.219.49|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2754 (2.7K) [application/x-gzip]
Saving to: ‘alibaba-cloud-log.tgz’

alibaba-cloud-log.tgz                              100%[================================================================================================================>]   2.69K  --. -KB/s    in 0s

2018-09-28 15:25:34 (79.6 MB/s) - ‘alibaba-cloud-log.tgz’ saved [2754/2754]

[INFO] your k8s is using project : k8s-log-c77a92ec5a3ce4e64a1bf13bde1820106
NAME:   alibaba-log-controller
LAST DEPLOYED: Fri Sep 28 15:25:34 2018
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1beta1/CustomResourceDefinition
NAME                                   AGE
aliyunlogconfigs.log.alibabacloud.com  0s

==> v1beta1/ClusterRole
alibaba-log-controller  0s

==> v1beta1/ClusterRoleBinding
NAME                    AGE
alibaba-log-controller  0s

==> v1beta1/DaemonSet
NAME        DESIRED  CURRENT  READY  UP-TO-DATE  AVAILABLE  NODE SELECTOR  AGE
logtail-ds  2        2        0      2           0          <none>         0s

==> v1beta1/Deployment
NAME                    DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
alibaba-log-controller  1        1        1           0          0s

==> v1/Pod(related)
NAME                                     READY  STATUS             RESTARTS  AGE
logtail-ds-6v979                         0/1    ContainerCreating  0         0s
logtail-ds-7ccqv                         0/1    ContainerCreating  0         0s
alibaba-log-controller-84d8b6b8cf-nkrkx  0/1    ContainerCreating  0         0s

==> v1/ServiceAccount
NAME                    SECRETS  AGE
alibaba-log-controller  1        0s


[SUCCESS] install helm package : alibaba-log-controller success.

```

## Upgrade Log Service components {#section_b3f_y5r_gfb .section}

If you have installed Log Service components of an early version through Helm or DaemonSet, upgrade or migrate the components as follows.

**Note:** To perform the following operations, first log on to the master node of your Kubernetes cluster of Alibaba Cloud Container Service. For information about the logon method, see [Connect to a Kubernetes cluster by using kubectl](reseller.en-US/User Guide/Kubernetes cluster/Cluster management/Connect to a Kubernetes cluster by using kubectl.md#).

**Use Helm to upgrade the components \(recommended\)**

1.  Run the following command to download the latest Helm package of Log Service components:

    ```
    wget http://logtail-release-cn-hangzhou.oss-cn-hangzhou.aliyuncs.com/kubernetes/alibaba-cloud-log.tgz -O alibaba-cloud-log.tgz
    
    ```

2.  Upgrade the components by using helm upgrade. The command is as follows:

    ```
    helm get values alibaba-log-controller --all > values.yaml && helm upgrade alibaba-log-controller alibaba-cloud-log.tgz --recreate-pods -f values.yaml
    
    ```


## DaemonSet migrate {#section_e3f_y5r_gfb .section}

This upgrade method is applicable to the situation that you find the components are installed through the old DaemonSet when you **check the Log Service component version**. This method does not support configuring Log Service in Container Service. You can upgrade the components as follows:

1.  At the end of the installation command, add a parameter which is the name of the project of Log Service used by your Kubernetes cluster.

    For example, if the project name is k8s-log-demo and the cluster ID is c12ba2028cxxxxxxxxxx6939f0b, then the installation command is:

    ```
    wget https://acs-logging.oss-cn-hangzhou.aliyuncs.com/alicloud-k8s-log-installer.sh -O alicloud-k8s-log-installer.sh; chmod 744 ./alicloud-k8s-log-installer.sh; ./alicloud-k8s-log-installer.sh --cluster-id c12ba2028cxxxxxxxxxx6939f0b --ali-uid 19*********19 --region-id cn-hangzhou --log-project k8s-log-demo
    
    ```

2.  After you complete the installation, log on the [Log Service console](https://partners-intl.console.aliyun.com/#/sls).
3.  In the Log service console, apply the history collection configuration of the project and Logstore to the new machine group `k8s-group-${your_k8s_cluster_id}`.
4.  After one minute, the history collection configuration is unbound from the history machine group.
5.  When log collection is normal, you can delete the previously installed Logtail DaemonSet.

**Note:** During the upgrade, some logs are duplicated. The CRD configuration management takes effect only for the configuration created by using CRD. The history configuration does not support the CRD management because the history configuration is created by using the non-CRD mode.

## Configure Log Service when creating an application {#section_g3f_y5r_gfb .section}

Container Service allows you to configure Log Service to collect container logs when creating an application. Currently, you can use the console or a YAML template to create an application.

**Create an application by using the console**

1.  Log on to the [Container Service console](https://partners-intl.console.aliyun.com/#/cs).
2.  Under the Kubernetes menu, click **Application** \> **Deployment** in the left-side navigation pane, and then click **Create by Image** in the upper-right corner.
3.  Configure **Name**, **Cluster**, **Namespace**, **Replicas**, and **Type**, and then click **Next**.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/17400/15474591669451_en-US.png)

4.  On the Container page, select the Tomcat image and configure container log collection.

    The following describes only configurations related to Log Service. For information about other application configurations, see [Create a deployment application by using an image](reseller.en-US/User Guide/Kubernetes cluster/Application management/Create a deployment application by using an image.md#).

5.  Configure **Log Service**. Click the **+** sign to create a configuration which consists of a **Logstore name** and a **log path**.

    -   Logstore name: Specify a Logstore in which collected logs are stored. If your specified Logstore does not exist, the system automatically creates the Logstore in the project of Log Service with which the cluster is associated .

        **Note:** A Logstore name cannot contain underscores \(\_\). You can use hyphens \(-\) instead.

    -   Log path: Specify the path where logs to be collected reside. For example, use /usr/local/tomcat/logs/catalina. \*.log to collect text logs of tomcat.

        **Note:** If you specify the log path as stdout, the container standard output and standard error output will be collected.

        Each configuration is automatically created as a configuration for the corresponding Logstore. By default, the simple mode \(by row\) is used to collect logs. To use more collection modes, log on go to the Log Service console, and enter the corresponding project \(prefixed with k8s-log by default\) and Logstore to modify the configuration.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/17400/15474591669460_en-US.png)

6.  Custom tag. Click the **+** sign to create a new custom tag. Each custom tag is a key-value pair which will be added to collected logs. You can use a custom tag to mark container logs. For example, you can create a custom tag as a version number.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/17400/15474591669473_en-US.png)

7.  When you complete all the configurations of the container, click **Next** in the upper-right corner to perform further configurations. For more information, see [Create a deployment application by using an image](reseller.en-US/User Guide/Kubernetes cluster/Application management/Create a deployment application by using an image.md#).

**Create an application by using a YAML template**

1.  Log on to the [Container Service console](https://partners-intl.console.aliyun.com/#/cs).
2.  Under the Kubernetes menu, click **Application** \> **Deployment** in the left-side navigation pane, and then click **Create by Template** in the upper-right corner.
3.  The syntax of the YAML template is the same as the Kubernetes syntax. To specify the collection configuration for the container, you need to use env to add **collection configuration** and **custom tag** for the container, and create corresponding volumeMounts and volumns. The following is a simple pod example:

    ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: my-demo
    spec:
      containers:
      - name: my-demo-app
        image: 'registry.cn-hangzhou.aliyuncs.com/log-service/docker-log-test:latest'
        env:
        ######### Configure environment variables  ###########
        - name: aliyun_logs_log-stdout
          value: stdout
        - name: aliyun_logs_log-varlog
          value: /var/log/*.log
        - name: aliyun_logs_mytag1_tags
          value: tag1=v1
        ###############################
        ######### Configure vulume mount ###########
        volumeMounts:
        - name: volumn-sls-mydemo
          mountPath: /var/log
      volumes:
      - name: volumn-sls-mydemo
        emptyDir: {}
      ###############################
    ```

    -   Configure three parts in order based on your needs.
    -   In the first part, use environment variables to create your **collection configuration** and **custom tag**. All environment variables related to configuration are prefixed with `aliyun_logs_`.
    -   Rules for creating the collection configuration are as follows:

        ```
        - name: aliyun_logs_{Logstore name}
          value: {log path}
        
        ```

        In the example, create two collection configurations. The `aliyun_logs_log-stdout` env creates a configuration that contains a Logstore named log-stdout and the log path of stdout. The standard output of the container is collected and stored to the Logstore named log-stdout.

        **Note:** A Logstore name cannot contain underscores \(\_\). You can use hyphens \(-\) instead.

    -   Rules for creating **a custom tag** are as follows:

        ```
        - name: aliyun_logs_{a name without '_'}_tags
          value: {Tag name}={Tag value}
        
        ```

        After a tag is configured, when logs of the container are collected, fields corresponding to the tag are automatically attached to Log Service.

    -   If you specify a non-stdout log path in your collection configuration, create corresponding volumnMounts in this part.

        In the example, the /var/log/\*.log log path is added to the collection configuration, therefore, the /var/log volumeMounts is added.

4.  When you complete a YAML template, click **DEPLOY** to deliver the configurations in the template to the Kubernetes cluster to execute.

## View logs {#section_t3f_y5r_gfb .section}

In this example, view logs of the tomcat application created in the console. After you complete the application configuration, logs of the tomcat application are collected and stored to Log Service. You can view your logs as follows:

1.  Log on to the [Log Service console](https://partners-intl.console.aliyun.com/#/sls).
2.  In the console, select the project \(k8s-log-\{Kubernetes cluster ID\} by default\) corresponding to the Kubernetes cluster.
3.  In the Logstore list, locate the Logstore specified in your configuration and click **Search**.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/17400/15474591669474_en-US.png)

4.  In this example, on the log search page, you can view the standard output logs of the tomcat application and text logs in the container, and you can find your custom tag is attached to log fields.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/17400/15474591669541_en-US.png)


## More information {#section_rg2_tdh_hfb .section}

1.  By default, the system use the simple mode to collect your data, that is, to collect data by row without parsing. To perform more complex configurations, see the following Log Service documents and log on to the Log Service console to modify configurations.
    -   [Container text logs](../../../../../reseller.en-US/User Guide/Logtail collection/Container log collection/Container text logs.md#)
    -   [Containers-standard output](../../../../../reseller.en-US/User Guide/Logtail collection/Container log collection/Containers-standard output.md#)
2.  Currently, Log Service uses plug-ins to collect the standard output logs of containers. You can configure more plug-ins to process collected logs further, such as to filter and extract fields.
3.  In addition to configuring log collection through the console, you can also directly collect logs of the Kubernetes cluster through the CRD configuration. For more information, see [Configure Kubernetes log collection on CRD](../../../../../reseller.en-US/User Guide/Logtail collection/Container log collection/Configure Kubernetes log collection on CRD.md#).
4.  For troubleshooting exceptions, see [Troubleshoot collection errors](../../../../../reseller.en-US/FAQ/Log collection/Troubleshoot collection errors.md#).
