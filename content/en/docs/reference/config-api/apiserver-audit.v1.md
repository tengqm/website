




---
title: audit.k8s.io/v1
content_type: tool-reference
---


## Resource Types 


  
- [Event](#audit-k8s-io-v1-Event)
- [EventList](#audit-k8s-io-v1-EventList)
- [Policy](#audit-k8s-io-v1-Policy)
- [PolicyList](#audit-k8s-io-v1-PolicyList)
  
    


## `Event`     {#audit-k8s-io-v1-Event}
    



**Appears in:**

- [EventList](#audit-k8s-io-v1-EventList)


Event captures all the information that can be included in an API audit log.

<table class="table">
<thead><tr><th width="30%">Field</th><th>Description</th></tr></thead>
<tbody>
    
<tr><td><samp>apiVersion</samp><br/>string</td><td><samp>audit.k8s.io/v1</samp></td></tr>
<tr><td><samp>kind</samp><br/>string</td><td><samp>Event</samp></td></tr>
    

  
  
<tr><td><samp>level</samp> <B>[Required]</B><br/>
<a href="#audit-k8s-io-v1-Level"><samp>Level</samp></a>
</td>
<td>
   AuditLevel at which event was generated</td>
</tr>
    
  
<tr><td><samp>auditID</samp> <B>[Required]</B><br/>
<a href="https://godoc.org/k8s.io/apimachinery/pkg/types#UID"><samp>k8s.io/apimachinery/pkg/types.UID</samp></a>
</td>
<td>
   Unique audit ID, generated for each request.</td>
</tr>
    
  
<tr><td><samp>stage</samp> <B>[Required]</B><br/>
<a href="#audit-k8s-io-v1-Stage"><samp>Stage</samp></a>
</td>
<td>
   Stage of the request handling when this event instance was generated.</td>
</tr>
    
  
<tr><td><samp>requestURI</samp> <B>[Required]</B><br/>
<samp>string</samp>
</td>
<td>
   RequestURI is the request URI as sent by the client to a server.</td>
</tr>
    
  
<tr><td><samp>verb</samp> <B>[Required]</B><br/>
<samp>string</samp>
</td>
<td>
   Verb is the kubernetes verb associated with the request.
For non-resource requests, this is the lower-cased HTTP method.</td>
</tr>
    
  
<tr><td><samp>user</samp> <B>[Required]</B><br/>
<a href="https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.19/#userinfo-v1-authentication"><samp>authentication/v1.UserInfo</samp></a>
</td>
<td>
   Authenticated user information.</td>
</tr>
    
  
<tr><td><samp>impersonatedUser</samp><br/>
<a href="https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.19/#userinfo-v1-authentication"><samp>authentication/v1.UserInfo</samp></a>
</td>
<td>
   Impersonated user information.</td>
</tr>
    
  
<tr><td><samp>sourceIPs</samp><br/>
<samp>[]string</samp>
</td>
<td>
   Source IPs, from where the request originated and intermediate proxies.</td>
</tr>
    
  
<tr><td><samp>userAgent</samp><br/>
<samp>string</samp>
</td>
<td>
   UserAgent records the user agent string reported by the client.
Note that the UserAgent is provided by the client, and must not be trusted.</td>
</tr>
    
  
<tr><td><samp>objectRef</samp><br/>
<a href="#audit-k8s-io-v1-ObjectReference"><samp>ObjectReference</samp></a>
</td>
<td>
   Object reference this request is targeted at.
Does not apply for List-type requests, or non-resource requests.</td>
</tr>
    
  
<tr><td><samp>responseStatus</samp><br/>
<a href="https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.19/#status-v1-meta"><samp>meta/v1.Status</samp></a>
</td>
<td>
   The response status, populated even when the ResponseObject is not a Status type.
For successful responses, this will only include the Code and StatusSuccess.
For non-status type error responses, this will be auto-populated with the error Message.</td>
</tr>
    
  
<tr><td><samp>requestObject</samp><br/>
<a href="https://godoc.org/k8s.io/apimachinery/pkg/runtime#Unknown"><samp>k8s.io/apimachinery/pkg/runtime.Unknown</samp></a>
</td>
<td>
   API object from the request, in JSON format. The RequestObject is recorded as-is in the request
(possibly re-encoded as JSON), prior to version conversion, defaulting, admission or
merging. It is an external versioned object type, and may not be a valid object on its own.
Omitted for non-resource requests.  Only logged at Request Level and higher.</td>
</tr>
    
  
<tr><td><samp>responseObject</samp><br/>
<a href="https://godoc.org/k8s.io/apimachinery/pkg/runtime#Unknown"><samp>k8s.io/apimachinery/pkg/runtime.Unknown</samp></a>
</td>
<td>
   API object returned in the response, in JSON. The ResponseObject is recorded after conversion
to the external type, and serialized as JSON.  Omitted for non-resource requests.  Only logged
at Response Level.</td>
</tr>
    
  
<tr><td><samp>requestReceivedTimestamp</samp><br/>
<a href="https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.19/#microtime-v1-meta"><samp>meta/v1.MicroTime</samp></a>
</td>
<td>
   Time the request reached the apiserver.</td>
</tr>
    
  
<tr><td><samp>stageTimestamp</samp><br/>
<a href="https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.19/#microtime-v1-meta"><samp>meta/v1.MicroTime</samp></a>
</td>
<td>
   Time the request reached current audit stage.</td>
</tr>
    
  
<tr><td><samp>annotations</samp><br/>
<samp>map[string]string</samp>
</td>
<td>
   Annotations is an unstructured key value map stored with an audit event that may be set by
plugins invoked in the request serving chain, including authentication, authorization and
admission plugins. Note that these annotations are for the audit event, and do not correspond
to the metadata.annotations of the submitted object. Keys should uniquely identify the informing
component to avoid name collisions (e.g. podsecuritypolicy.admission.k8s.io/policy). Values
should be short. Annotations are included in the Metadata level.</td>
</tr>
    
  
</tbody>
</table>
    


## `EventList`     {#audit-k8s-io-v1-EventList}
    




EventList is a list of audit Events.

<table class="table">
<thead><tr><th width="30%">Field</th><th>Description</th></tr></thead>
<tbody>
    
<tr><td><samp>apiVersion</samp><br/>string</td><td><samp>audit.k8s.io/v1</samp></td></tr>
<tr><td><samp>kind</samp><br/>string</td><td><samp>EventList</samp></td></tr>
    

  
  
<tr><td><samp>metadata</samp><br/>
<a href="https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.19/#listmeta-v1-meta"><samp>meta/v1.ListMeta</samp></a>
</td>
<td>
   <span class="text-muted">No description provided.</span>
   </td>
</tr>
    
  
<tr><td><samp>items</samp> <B>[Required]</B><br/>
<a href="#audit-k8s-io-v1-Event"><samp>[]Event</samp></a>
</td>
<td>
   <span class="text-muted">No description provided.</span>
   </td>
</tr>
    
  
</tbody>
</table>
    


## `Policy`     {#audit-k8s-io-v1-Policy}
    



**Appears in:**

- [PolicyList](#audit-k8s-io-v1-PolicyList)


Policy defines the configuration of audit logging, and the rules for how different request
categories are logged.

<table class="table">
<thead><tr><th width="30%">Field</th><th>Description</th></tr></thead>
<tbody>
    
<tr><td><samp>apiVersion</samp><br/>string</td><td><samp>audit.k8s.io/v1</samp></td></tr>
<tr><td><samp>kind</samp><br/>string</td><td><samp>Policy</samp></td></tr>
    

  
  
<tr><td><samp>metadata</samp><br/>
<a href="https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.19/#objectmeta-v1-meta"><samp>meta/v1.ObjectMeta</samp></a>
</td>
<td>
   ObjectMeta is included for interoperability with API infrastructure.Refer to the Kubernetes API documentation for the fields of the <samp>metadata</samp> field.</td>
</tr>
    
  
<tr><td><samp>rules</samp> <B>[Required]</B><br/>
<a href="#audit-k8s-io-v1-PolicyRule"><samp>[]PolicyRule</samp></a>
</td>
<td>
   Rules specify the audit Level a request should be recorded at.
A request may match multiple rules, in which case the FIRST matching rule is used.
The default audit level is None, but can be overridden by a catch-all rule at the end of the list.
PolicyRules are strictly ordered.</td>
</tr>
    
  
<tr><td><samp>omitStages</samp><br/>
<a href="#audit-k8s-io-v1-Stage"><samp>[]Stage</samp></a>
</td>
<td>
   OmitStages is a list of stages for which no events are created. Note that this can also
be specified per rule in which case the union of both are omitted.</td>
</tr>
    
  
</tbody>
</table>
    


## `PolicyList`     {#audit-k8s-io-v1-PolicyList}
    




PolicyList is a list of audit Policies.

<table class="table">
<thead><tr><th width="30%">Field</th><th>Description</th></tr></thead>
<tbody>
    
<tr><td><samp>apiVersion</samp><br/>string</td><td><samp>audit.k8s.io/v1</samp></td></tr>
<tr><td><samp>kind</samp><br/>string</td><td><samp>PolicyList</samp></td></tr>
    

  
  
<tr><td><samp>metadata</samp><br/>
<a href="https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.19/#listmeta-v1-meta"><samp>meta/v1.ListMeta</samp></a>
</td>
<td>
   <span class="text-muted">No description provided.</span>
   </td>
</tr>
    
  
<tr><td><samp>items</samp> <B>[Required]</B><br/>
<a href="#audit-k8s-io-v1-Policy"><samp>[]Policy</samp></a>
</td>
<td>
   <span class="text-muted">No description provided.</span>
   </td>
</tr>
    
  
</tbody>
</table>
    


## `GroupResources`     {#audit-k8s-io-v1-GroupResources}
    



**Appears in:**

- [PolicyRule](#audit-k8s-io-v1-PolicyRule)


GroupResources represents resource kinds in an API group.

<table class="table">
<thead><tr><th width="30%">Field</th><th>Description</th></tr></thead>
<tbody>
    

  
<tr><td><samp>group</samp><br/>
<samp>string</samp>
</td>
<td>
   Group is the name of the API group that contains the resources.
The empty string represents the core API group.</td>
</tr>
    
  
<tr><td><samp>resources</samp><br/>
<samp>[]string</samp>
</td>
<td>
   Resources is a list of resources this rule applies to.

For example:
'pods' matches pods.
'pods/log' matches the log subresource of pods.
'&lowast;' matches all resources and their subresources.
'pods/&lowast;' matches all subresources of pods.
'&lowast;/scale' matches all scale subresources.

If wildcard is present, the validation rule will ensure resources do not
overlap with each other.

An empty list implies all resources and subresources in this API groups apply.</td>
</tr>
    
  
<tr><td><samp>resourceNames</samp><br/>
<samp>[]string</samp>
</td>
<td>
   ResourceNames is a list of resource instance names that the policy matches.
Using this field requires Resources to be specified.
An empty list implies that every instance of the resource is matched.</td>
</tr>
    
  
</tbody>
</table>
    


## `Level`     {#audit-k8s-io-v1-Level}
    
(Alias of `string`)


**Appears in:**

- [Event](#audit-k8s-io-v1-Event)

- [PolicyRule](#audit-k8s-io-v1-PolicyRule)


Level defines the amount of information logged during auditing


    


## `ObjectReference`     {#audit-k8s-io-v1-ObjectReference}
    



**Appears in:**

- [Event](#audit-k8s-io-v1-Event)


ObjectReference contains enough information to let you inspect or modify the referred object.

<table class="table">
<thead><tr><th width="30%">Field</th><th>Description</th></tr></thead>
<tbody>
    

  
<tr><td><samp>resource</samp><br/>
<samp>string</samp>
</td>
<td>
   <span class="text-muted">No description provided.</span>
   </td>
</tr>
    
  
<tr><td><samp>namespace</samp><br/>
<samp>string</samp>
</td>
<td>
   <span class="text-muted">No description provided.</span>
   </td>
</tr>
    
  
<tr><td><samp>name</samp><br/>
<samp>string</samp>
</td>
<td>
   <span class="text-muted">No description provided.</span>
   </td>
</tr>
    
  
<tr><td><samp>uid</samp><br/>
<a href="https://godoc.org/k8s.io/apimachinery/pkg/types#UID"><samp>k8s.io/apimachinery/pkg/types.UID</samp></a>
</td>
<td>
   <span class="text-muted">No description provided.</span>
   </td>
</tr>
    
  
<tr><td><samp>apiGroup</samp><br/>
<samp>string</samp>
</td>
<td>
   APIGroup is the name of the API group that contains the referred object.
The empty string represents the core API group.</td>
</tr>
    
  
<tr><td><samp>apiVersion</samp><br/>
<samp>string</samp>
</td>
<td>
   APIVersion is the version of the API group that contains the referred object.</td>
</tr>
    
  
<tr><td><samp>resourceVersion</samp><br/>
<samp>string</samp>
</td>
<td>
   <span class="text-muted">No description provided.</span>
   </td>
</tr>
    
  
<tr><td><samp>subresource</samp><br/>
<samp>string</samp>
</td>
<td>
   <span class="text-muted">No description provided.</span>
   </td>
</tr>
    
  
</tbody>
</table>
    


## `PolicyRule`     {#audit-k8s-io-v1-PolicyRule}
    



**Appears in:**

- [Policy](#audit-k8s-io-v1-Policy)


PolicyRule maps requests based off metadata to an audit Level.
Requests must match the rules of every field (an intersection of rules).

<table class="table">
<thead><tr><th width="30%">Field</th><th>Description</th></tr></thead>
<tbody>
    

  
<tr><td><samp>level</samp> <B>[Required]</B><br/>
<a href="#audit-k8s-io-v1-Level"><samp>Level</samp></a>
</td>
<td>
   The Level that requests matching this rule are recorded at.</td>
</tr>
    
  
<tr><td><samp>users</samp><br/>
<samp>[]string</samp>
</td>
<td>
   The users (by authenticated user name) this rule applies to.
An empty list implies every user.</td>
</tr>
    
  
<tr><td><samp>userGroups</samp><br/>
<samp>[]string</samp>
</td>
<td>
   The user groups this rule applies to. A user is considered matching
if it is a member of any of the UserGroups.
An empty list implies every user group.</td>
</tr>
    
  
<tr><td><samp>verbs</samp><br/>
<samp>[]string</samp>
</td>
<td>
   The verbs that match this rule.
An empty list implies every verb.</td>
</tr>
    
  
<tr><td><samp>resources</samp><br/>
<a href="#audit-k8s-io-v1-GroupResources"><samp>[]GroupResources</samp></a>
</td>
<td>
   Resources that this rule matches. An empty list implies all kinds in all API groups.</td>
</tr>
    
  
<tr><td><samp>namespaces</samp><br/>
<samp>[]string</samp>
</td>
<td>
   Namespaces that this rule matches.
The empty string "" matches non-namespaced resources.
An empty list implies every namespace.</td>
</tr>
    
  
<tr><td><samp>nonResourceURLs</samp><br/>
<samp>[]string</samp>
</td>
<td>
   NonResourceURLs is a set of URL paths that should be audited.
&lowast;s are allowed, but only as the full, final step in the path.
Examples:
 "/metrics" - Log requests for apiserver metrics
 "/healthz&lowast;" - Log all health checks</td>
</tr>
    
  
<tr><td><samp>omitStages</samp><br/>
<a href="#audit-k8s-io-v1-Stage"><samp>[]Stage</samp></a>
</td>
<td>
   OmitStages is a list of stages for which no events are created. Note that this can also
be specified policy wide in which case the union of both are omitted.
An empty list means no restrictions will apply.</td>
</tr>
    
  
</tbody>
</table>
    


## `Stage`     {#audit-k8s-io-v1-Stage}
    
(Alias of `string`)


**Appears in:**

- [Event](#audit-k8s-io-v1-Event)

- [Policy](#audit-k8s-io-v1-Policy)

- [PolicyRule](#audit-k8s-io-v1-PolicyRule)


Stage defines the stages in request handling that audit events may be generated.


    
  
