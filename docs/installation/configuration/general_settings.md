# General settings

Assemblyline come built in with sensible defaults, there are however some values which might require tweaking for your environment.

Your configuration file location will depend on your deployment type:

<table>
<tr>
<td style="background-color:#009c7b"><text style="color:white;">Appliance deployment</text></td>
<td> edit `./config/config.yml` </td>
</tr>
<tr>
<td style="background-color:#2869e6"><text style="color:white;">Cluster deployment</text></td>
<td> see <a href="https://github.com/CybercentreCanada/assemblyline-helm-chart/blob/master/assemblyline/values.yaml"> Default helm chart</a> </td>
</tr>
</table>

<hr>

## max submission size
Maximum file size that can be submitted in the system

Appliance(docker)
{: .label .label-green }
    ingressAnnotations:
        kubernetes.io/ingress.class: "nginx"
        ginx.ingress.kubernetes.io/proxy-body-size: 100M

Cluster(k8s)
{: .label .label-blue }
    ingressAnnotations:
        kubernetes.io/ingress.class: "nginx"
        ginx.ingress.kubernetes.io/proxy-body-size: 100M


## time to live
Amount of time (in days) that a file will stay in the system

Appliance(docker)
{: .label .label-green }
    configuration:
        submission:
            dtl: 30

Cluster(k8s)
{: .label .label-blue }
    configuration:
        submission:
            dtl: 30

## term of services (banner)
Customized user banner with terms of use for the system

Appliance(docker)
{: .label .label-green }
    ui:
        tos: | 
            ### *Terms of services*

Cluster(k8s)
{: .label .label-blue }
    ui:
        tos: | 
            ### *Terms of services*