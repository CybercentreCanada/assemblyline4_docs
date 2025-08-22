# Kubernetes

This section contains troubleshooting steps for deploying Assemblyline in a Kubernetes environment.

??? question "Connection timeouts to external domains"
    By default, coreDNS is configured to the Google's Public DNS when trying to resolve external domains outside the cluster.

    You can configure it to use a different DNS via:
    ```sudo microk8s disable dns && sudo microk8s enabled dns:1.1.1.1```

    If this still poses an issue, refer to: [Teleport - Troubleshooting Kubernetes Networking Issues](https://goteleport.com/blog/troubleshooting-kubernetes-networking/)
??? question "How can I monitor the status of the deployment/cluster?"
    The quickest way to monitor the status of your cluster is:
    ```kubectl get pods -n <assemblyline_namespace>```

    There are other tools such as [k9s](https://k9scli.io/) and [FreeLens](https://github.com/freelensapp/freelens) (an OSS fork of [Lens](https://k8slens.dev/)/[OpenLens](https://github.com/lensapp/lens)) that allow you to monitor your cluster in a more user-friendly manner.
??? question "Are HPAs adjustable?"
    Depending on the amount of activity you're receiving, you'll likely have to tweak the *TargetUsage and*ReqRam settings in your values.yaml for your particular deployment, either to cause it to scale faster or slower.

    Refer to: [Kubernetes - Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) for more information.
??? question "NGINX 504 but everything seems to be running"
    It's possible the domain you're accessing the UI with doesn't match the setting ```configuration.ui.fqdn``` in your values.yaml.
    If your setting is set to 'localhost' but you're accessing the UI using '192.0.0.1.nip.io', there is no ingress path using '192.0.0.1.nip.io' as a base.

    The simplest solution is to update your values.yaml to the appropriate FQDN and redeploy.
??? question "Is it possible to mount an internal root CA bundle into core components to use?"
    Yes! This would involve creating a configmap containing your CA bundle and using ```coreVolumes, coreMounts, and coreEnv``` in your values.yaml to pass that information onto the core deployments.

    An example of this configuration would be:
    ```yaml
    coreEnv:
        - name: REQUESTS_CA_BUNDLE
        value: /usr/share/internal-ca
    coreMounts:
        - name: internal-certs
        mountPath: /usr/share/internal-ca
    coreVolumes:
        - name: internal-certs
        configMap:
            name: internal-certs-config
    ```
??? question "Ignoring ingress because of error while validating ingress class" ingress="<namespace\>/<release\>-ingress" error="no object matching key "nginx" in local store"
    This error can typically happen if the Assemblyline ingress' class name doesn't match the class name of the ingress controller on the system. You can fix this by
    assigning the name of the class to `ingressClassName` in your values.yaml and re-deploy.

??? question "Service pods aren't getting scheduled because there are labels missing"
    You can add global labels to service pods by using the `core.scaler.additional_labels` or `core.scaler.privileged_services_additional_labels`:
    ```yaml
    core:
      scaler:
        additional_labels:
          - "service_section=normal"
        privileged_services_additional_labels:
          - "service_section=privileged"
    ```

    Alternatively, you can set labels on an individual container basis by using the UI/API (1)
    { .annotate }

    1.  ![Add a label to service containers](./images/add_container_label.png)
