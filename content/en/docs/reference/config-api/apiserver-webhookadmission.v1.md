




---
title: apiserver.config.k8s.io/v1
content_type: tool-reference
---
Package v1 is the v1 version of the API.

## Resource Types 


  
- [WebhookAdmission](#apiserver-config-k8s-io-v1-WebhookAdmission)
  
    


## `WebhookAdmission`     {#apiserver-config-k8s-io-v1-WebhookAdmission}
    




WebhookAdmission provides configuration for the webhook admission controller.

<table class="table">
<thead><tr><th width="30%">Field</th><th>Description</th></tr></thead>
<tbody>
    
<tr><td><samp>apiVersion</samp><br/>string</td><td><samp>apiserver.config.k8s.io/v1</samp></td></tr>
<tr><td><samp>kind</samp><br/>string</td><td><samp>WebhookAdmission</samp></td></tr>
    

  
  
<tr><td><samp>kubeConfigFile</samp> <B>[Required]</B><br/>
<samp>string</samp>
</td>
<td>
   KubeConfigFile is the path to the kubeconfig file.</td>
</tr>
    
  
</tbody>
</table>
    
  
