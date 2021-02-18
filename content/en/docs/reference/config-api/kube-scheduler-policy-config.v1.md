




---
title: kubescheduler.config.k8s.io/v1
content_type: tool-reference
---


## Resource Types 


  
- [Policy](#kubescheduler-config-k8s-io-v1-Policy)
  
    


## `Policy`     {#kubescheduler-config-k8s-io-v1-Policy}
    




Policy describes a struct for a policy resource used in api.

<table class="table">
<thead><tr><th width="30%">Field</th><th>Description</th></tr></thead>
<tbody>
    
<tr><td><samp>apiVersion</samp><br/>string</td><td><samp>kubescheduler.config.k8s.io/v1</samp></td></tr>
<tr><td><samp>kind</samp><br/>string</td><td><samp>Policy</samp></td></tr>
    

  
  
<tr><td><samp>predicates</samp> <B>[Required]</B><br/>
<a href="#kubescheduler-config-k8s-io-v1-PredicatePolicy"><samp>[]PredicatePolicy</samp></a>
</td>
<td>
   Holds the information to configure the fit predicate functions</td>
</tr>
    
  
<tr><td><samp>priorities</samp> <B>[Required]</B><br/>
<a href="#kubescheduler-config-k8s-io-v1-PriorityPolicy"><samp>[]PriorityPolicy</samp></a>
</td>
<td>
   Holds the information to configure the priority functions</td>
</tr>
    
  
<tr><td><samp>extenders</samp> <B>[Required]</B><br/>
<a href="#kubescheduler-config-k8s-io-v1-LegacyExtender"><samp>[]LegacyExtender</samp></a>
</td>
<td>
   Holds the information to communicate with the extender(s)</td>
</tr>
    
  
<tr><td><samp>hardPodAffinitySymmetricWeight</samp> <B>[Required]</B><br/>
<samp>int32</samp>
</td>
<td>
   RequiredDuringScheduling affinity is not symmetric, but there is an implicit PreferredDuringScheduling affinity rule
corresponding to every RequiredDuringScheduling affinity rule.
HardPodAffinitySymmetricWeight represents the weight of implicit PreferredDuringScheduling affinity rule, in the range 1-100.</td>
</tr>
    
  
<tr><td><samp>alwaysCheckAllPredicates</samp> <B>[Required]</B><br/>
<samp>bool</samp>
</td>
<td>
   When AlwaysCheckAllPredicates is set to true, scheduler checks all
the configured predicates even after one or more of them fails.
When the flag is set to false, scheduler skips checking the rest
of the predicates after it finds one predicate that failed.</td>
</tr>
    
  
</tbody>
</table>
    


## `ExtenderManagedResource`     {#kubescheduler-config-k8s-io-v1-ExtenderManagedResource}
    



**Appears in:**

- [Extender](#kubescheduler-config-k8s-io-v1beta1-Extender)

- [LegacyExtender](#kubescheduler-config-k8s-io-v1-LegacyExtender)

- [LegacyExtender](#kubescheduler-config-k8s-io-v1-LegacyExtender)


ExtenderManagedResource describes the arguments of extended resources
managed by an extender.

<table class="table">
<thead><tr><th width="30%">Field</th><th>Description</th></tr></thead>
<tbody>
    

  
<tr><td><samp>name</samp> <B>[Required]</B><br/>
<samp>string</samp>
</td>
<td>
   Name is the extended resource name.</td>
</tr>
    
  
<tr><td><samp>ignoredByScheduler</samp> <B>[Required]</B><br/>
<samp>bool</samp>
</td>
<td>
   IgnoredByScheduler indicates whether kube-scheduler should ignore this
resource when applying predicates.</td>
</tr>
    
  
</tbody>
</table>
    


## `ExtenderTLSConfig`     {#kubescheduler-config-k8s-io-v1-ExtenderTLSConfig}
    



**Appears in:**

- [Extender](#kubescheduler-config-k8s-io-v1beta1-Extender)

- [LegacyExtender](#kubescheduler-config-k8s-io-v1-LegacyExtender)

- [LegacyExtender](#kubescheduler-config-k8s-io-v1-LegacyExtender)


ExtenderTLSConfig contains settings to enable TLS with extender

<table class="table">
<thead><tr><th width="30%">Field</th><th>Description</th></tr></thead>
<tbody>
    

  
<tr><td><samp>insecure</samp> <B>[Required]</B><br/>
<samp>bool</samp>
</td>
<td>
   Server should be accessed without verifying the TLS certificate. For testing only.</td>
</tr>
    
  
<tr><td><samp>serverName</samp> <B>[Required]</B><br/>
<samp>string</samp>
</td>
<td>
   ServerName is passed to the server for SNI and is used in the client to check server
certificates against. If ServerName is empty, the hostname used to contact the
server is used.</td>
</tr>
    
  
<tr><td><samp>certFile</samp> <B>[Required]</B><br/>
<samp>string</samp>
</td>
<td>
   Server requires TLS client certificate authentication</td>
</tr>
    
  
<tr><td><samp>keyFile</samp> <B>[Required]</B><br/>
<samp>string</samp>
</td>
<td>
   Server requires TLS client certificate authentication</td>
</tr>
    
  
<tr><td><samp>caFile</samp> <B>[Required]</B><br/>
<samp>string</samp>
</td>
<td>
   Trusted root certificates for server</td>
</tr>
    
  
<tr><td><samp>certData</samp> <B>[Required]</B><br/>
<samp>[]byte</samp>
</td>
<td>
   CertData holds PEM-encoded bytes (typically read from a client certificate file).
CertData takes precedence over CertFile</td>
</tr>
    
  
<tr><td><samp>keyData</samp> <B>[Required]</B><br/>
<samp>[]byte</samp>
</td>
<td>
   KeyData holds PEM-encoded bytes (typically read from a client certificate key file).
KeyData takes precedence over KeyFile</td>
</tr>
    
  
<tr><td><samp>caData</samp> <B>[Required]</B><br/>
<samp>[]byte</samp>
</td>
<td>
   CAData holds PEM-encoded bytes (typically read from a root certificates bundle).
CAData takes precedence over CAFile</td>
</tr>
    
  
</tbody>
</table>
    


## `LabelPreference`     {#kubescheduler-config-k8s-io-v1-LabelPreference}
    



**Appears in:**

- [PriorityArgument](#kubescheduler-config-k8s-io-v1-PriorityArgument)

- [PriorityArgument](#kubescheduler-config-k8s-io-v1-PriorityArgument)


LabelPreference holds the parameters that are used to configure the corresponding priority function

<table class="table">
<thead><tr><th width="30%">Field</th><th>Description</th></tr></thead>
<tbody>
    

  
<tr><td><samp>label</samp> <B>[Required]</B><br/>
<samp>string</samp>
</td>
<td>
   Used to identify node "groups"</td>
</tr>
    
  
<tr><td><samp>presence</samp> <B>[Required]</B><br/>
<samp>bool</samp>
</td>
<td>
   This is a boolean flag
If true, higher priority is given to nodes that have the label
If false, higher priority is given to nodes that do not have the label</td>
</tr>
    
  
</tbody>
</table>
    


## `LabelsPresence`     {#kubescheduler-config-k8s-io-v1-LabelsPresence}
    



**Appears in:**

- [PredicateArgument](#kubescheduler-config-k8s-io-v1-PredicateArgument)

- [PredicateArgument](#kubescheduler-config-k8s-io-v1-PredicateArgument)


LabelsPresence holds the parameters that are used to configure the corresponding predicate in scheduler policy configuration.

<table class="table">
<thead><tr><th width="30%">Field</th><th>Description</th></tr></thead>
<tbody>
    

  
<tr><td><samp>labels</samp> <B>[Required]</B><br/>
<samp>[]string</samp>
</td>
<td>
   The list of labels that identify node "groups"
All of the labels should be either present (or absent) for the node to be considered a fit for hosting the pod</td>
</tr>
    
  
<tr><td><samp>presence</samp> <B>[Required]</B><br/>
<samp>bool</samp>
</td>
<td>
   The boolean flag that indicates whether the labels should be present or absent from the node</td>
</tr>
    
  
</tbody>
</table>
    


## `LegacyExtender`     {#kubescheduler-config-k8s-io-v1-LegacyExtender}
    



**Appears in:**

- [Policy](#kubescheduler-config-k8s-io-v1-Policy)

- [Policy](#kubescheduler-config-k8s-io-v1-Policy)


LegacyExtender holds the parameters used to communicate with the extender. If a verb is unspecified/empty,
it is assumed that the extender chose not to provide that extension.

<table class="table">
<thead><tr><th width="30%">Field</th><th>Description</th></tr></thead>
<tbody>
    

  
<tr><td><samp>urlPrefix</samp> <B>[Required]</B><br/>
<samp>string</samp>
</td>
<td>
   URLPrefix at which the extender is available</td>
</tr>
    
  
<tr><td><samp>filterVerb</samp> <B>[Required]</B><br/>
<samp>string</samp>
</td>
<td>
   Verb for the filter call, empty if not supported. This verb is appended to the URLPrefix when issuing the filter call to extender.</td>
</tr>
    
  
<tr><td><samp>preemptVerb</samp> <B>[Required]</B><br/>
<samp>string</samp>
</td>
<td>
   Verb for the preempt call, empty if not supported. This verb is appended to the URLPrefix when issuing the preempt call to extender.</td>
</tr>
    
  
<tr><td><samp>prioritizeVerb</samp> <B>[Required]</B><br/>
<samp>string</samp>
</td>
<td>
   Verb for the prioritize call, empty if not supported. This verb is appended to the URLPrefix when issuing the prioritize call to extender.</td>
</tr>
    
  
<tr><td><samp>weight</samp> <B>[Required]</B><br/>
<samp>int64</samp>
</td>
<td>
   The numeric multiplier for the node scores that the prioritize call generates.
The weight should be a positive integer</td>
</tr>
    
  
<tr><td><samp>bindVerb</samp> <B>[Required]</B><br/>
<samp>string</samp>
</td>
<td>
   Verb for the bind call, empty if not supported. This verb is appended to the URLPrefix when issuing the bind call to extender.
If this method is implemented by the extender, it is the extender's responsibility to bind the pod to apiserver. Only one extender
can implement this function.</td>
</tr>
    
  
<tr><td><samp>enableHttps</samp> <B>[Required]</B><br/>
<samp>bool</samp>
</td>
<td>
   EnableHTTPS specifies whether https should be used to communicate with the extender</td>
</tr>
    
  
<tr><td><samp>tlsConfig</samp> <B>[Required]</B><br/>
<a href="#kubescheduler-config-k8s-io-v1-ExtenderTLSConfig"><samp>ExtenderTLSConfig</samp></a>
</td>
<td>
   TLSConfig specifies the transport layer security config</td>
</tr>
    
  
<tr><td><samp>httpTimeout</samp> <B>[Required]</B><br/>
<a href="https://godoc.org/time#Duration"><samp>time.Duration</samp></a>
</td>
<td>
   HTTPTimeout specifies the timeout duration for a call to the extender. Filter timeout fails the scheduling of the pod. Prioritize
timeout is ignored, k8s/other extenders priorities are used to select the node.</td>
</tr>
    
  
<tr><td><samp>nodeCacheCapable</samp> <B>[Required]</B><br/>
<samp>bool</samp>
</td>
<td>
   NodeCacheCapable specifies that the extender is capable of caching node information,
so the scheduler should only send minimal information about the eligible nodes
assuming that the extender already cached full details of all nodes in the cluster</td>
</tr>
    
  
<tr><td><samp>managedResources</samp><br/>
<a href="#kubescheduler-config-k8s-io-v1-ExtenderManagedResource"><samp>[]ExtenderManagedResource</samp></a>
</td>
<td>
   ManagedResources is a list of extended resources that are managed by
this extender.
- A pod will be sent to the extender on the Filter, Prioritize and Bind
  (if the extender is the binder) phases iff the pod requests at least
  one of the extended resources in this list. If empty or unspecified,
  all pods will be sent to this extender.
- If IgnoredByScheduler is set to true for a resource, kube-scheduler
  will skip checking the resource in predicates.</td>
</tr>
    
  
<tr><td><samp>ignorable</samp> <B>[Required]</B><br/>
<samp>bool</samp>
</td>
<td>
   Ignorable specifies if the extender is ignorable, i.e. scheduling should not
fail when the extender returns an error or is not reachable.</td>
</tr>
    
  
</tbody>
</table>
    


## `PredicateArgument`     {#kubescheduler-config-k8s-io-v1-PredicateArgument}
    



**Appears in:**

- [PredicatePolicy](#kubescheduler-config-k8s-io-v1-PredicatePolicy)

- [PredicatePolicy](#kubescheduler-config-k8s-io-v1-PredicatePolicy)


PredicateArgument represents the arguments to configure predicate functions in scheduler policy configuration.
Only one of its members may be specified

<table class="table">
<thead><tr><th width="30%">Field</th><th>Description</th></tr></thead>
<tbody>
    

  
<tr><td><samp>serviceAffinity</samp> <B>[Required]</B><br/>
<a href="#kubescheduler-config-k8s-io-v1-ServiceAffinity"><samp>ServiceAffinity</samp></a>
</td>
<td>
   The predicate that provides affinity for pods belonging to a service
It uses a label to identify nodes that belong to the same "group"</td>
</tr>
    
  
<tr><td><samp>labelsPresence</samp> <B>[Required]</B><br/>
<a href="#kubescheduler-config-k8s-io-v1-LabelsPresence"><samp>LabelsPresence</samp></a>
</td>
<td>
   The predicate that checks whether a particular node has a certain label
defined or not, regardless of value</td>
</tr>
    
  
</tbody>
</table>
    


## `PredicatePolicy`     {#kubescheduler-config-k8s-io-v1-PredicatePolicy}
    



**Appears in:**

- [Policy](#kubescheduler-config-k8s-io-v1-Policy)

- [Policy](#kubescheduler-config-k8s-io-v1-Policy)


PredicatePolicy describes a struct of a predicate policy.

<table class="table">
<thead><tr><th width="30%">Field</th><th>Description</th></tr></thead>
<tbody>
    

  
<tr><td><samp>name</samp> <B>[Required]</B><br/>
<samp>string</samp>
</td>
<td>
   Identifier of the predicate policy
For a custom predicate, the name can be user-defined
For the Kubernetes provided predicates, the name is the identifier of the pre-defined predicate</td>
</tr>
    
  
<tr><td><samp>argument</samp> <B>[Required]</B><br/>
<a href="#kubescheduler-config-k8s-io-v1-PredicateArgument"><samp>PredicateArgument</samp></a>
</td>
<td>
   Holds the parameters to configure the given predicate</td>
</tr>
    
  
</tbody>
</table>
    


## `PriorityArgument`     {#kubescheduler-config-k8s-io-v1-PriorityArgument}
    



**Appears in:**

- [PriorityPolicy](#kubescheduler-config-k8s-io-v1-PriorityPolicy)

- [PriorityPolicy](#kubescheduler-config-k8s-io-v1-PriorityPolicy)


PriorityArgument represents the arguments to configure priority functions in scheduler policy configuration.
Only one of its members may be specified

<table class="table">
<thead><tr><th width="30%">Field</th><th>Description</th></tr></thead>
<tbody>
    

  
<tr><td><samp>serviceAntiAffinity</samp> <B>[Required]</B><br/>
<a href="#kubescheduler-config-k8s-io-v1-ServiceAntiAffinity"><samp>ServiceAntiAffinity</samp></a>
</td>
<td>
   The priority function that ensures a good spread (anti-affinity) for pods belonging to a service
It uses a label to identify nodes that belong to the same "group"</td>
</tr>
    
  
<tr><td><samp>labelPreference</samp> <B>[Required]</B><br/>
<a href="#kubescheduler-config-k8s-io-v1-LabelPreference"><samp>LabelPreference</samp></a>
</td>
<td>
   The priority function that checks whether a particular node has a certain label
defined or not, regardless of value</td>
</tr>
    
  
<tr><td><samp>requestedToCapacityRatioArguments</samp> <B>[Required]</B><br/>
<a href="#kubescheduler-config-k8s-io-v1-RequestedToCapacityRatioArguments"><samp>RequestedToCapacityRatioArguments</samp></a>
</td>
<td>
   The RequestedToCapacityRatio priority function is parametrized with function shape.</td>
</tr>
    
  
</tbody>
</table>
    


## `PriorityPolicy`     {#kubescheduler-config-k8s-io-v1-PriorityPolicy}
    



**Appears in:**

- [Policy](#kubescheduler-config-k8s-io-v1-Policy)

- [Policy](#kubescheduler-config-k8s-io-v1-Policy)


PriorityPolicy describes a struct of a priority policy.

<table class="table">
<thead><tr><th width="30%">Field</th><th>Description</th></tr></thead>
<tbody>
    

  
<tr><td><samp>name</samp> <B>[Required]</B><br/>
<samp>string</samp>
</td>
<td>
   Identifier of the priority policy
For a custom priority, the name can be user-defined
For the Kubernetes provided priority functions, the name is the identifier of the pre-defined priority function</td>
</tr>
    
  
<tr><td><samp>weight</samp> <B>[Required]</B><br/>
<samp>int64</samp>
</td>
<td>
   The numeric multiplier for the node scores that the priority function generates
The weight should be non-zero and can be a positive or a negative integer</td>
</tr>
    
  
<tr><td><samp>argument</samp> <B>[Required]</B><br/>
<a href="#kubescheduler-config-k8s-io-v1-PriorityArgument"><samp>PriorityArgument</samp></a>
</td>
<td>
   Holds the parameters to configure the given priority function</td>
</tr>
    
  
</tbody>
</table>
    


## `RequestedToCapacityRatioArguments`     {#kubescheduler-config-k8s-io-v1-RequestedToCapacityRatioArguments}
    



**Appears in:**

- [PriorityArgument](#kubescheduler-config-k8s-io-v1-PriorityArgument)

- [PriorityArgument](#kubescheduler-config-k8s-io-v1-PriorityArgument)


RequestedToCapacityRatioArguments holds arguments specific to RequestedToCapacityRatio priority function.

<table class="table">
<thead><tr><th width="30%">Field</th><th>Description</th></tr></thead>
<tbody>
    

  
<tr><td><samp>shape</samp> <B>[Required]</B><br/>
<a href="#kubescheduler-config-k8s-io-v1-UtilizationShapePoint"><samp>[]UtilizationShapePoint</samp></a>
</td>
<td>
   Array of point defining priority function shape.</td>
</tr>
    
  
<tr><td><samp>resources</samp> <B>[Required]</B><br/>
<a href="#kubescheduler-config-k8s-io-v1-ResourceSpec"><samp>[]ResourceSpec</samp></a>
</td>
<td>
   <span class="text-muted">No description provided.</span>
   </td>
</tr>
    
  
</tbody>
</table>
    


## `ResourceSpec`     {#kubescheduler-config-k8s-io-v1-ResourceSpec}
    



**Appears in:**

- [RequestedToCapacityRatioArguments](#kubescheduler-config-k8s-io-v1-RequestedToCapacityRatioArguments)

- [RequestedToCapacityRatioArguments](#kubescheduler-config-k8s-io-v1-RequestedToCapacityRatioArguments)


ResourceSpec represents single resource and weight for bin packing of priority RequestedToCapacityRatioArguments.

<table class="table">
<thead><tr><th width="30%">Field</th><th>Description</th></tr></thead>
<tbody>
    

  
<tr><td><samp>name</samp> <B>[Required]</B><br/>
<samp>string</samp>
</td>
<td>
   Name of the resource to be managed by RequestedToCapacityRatio function.</td>
</tr>
    
  
<tr><td><samp>weight</samp> <B>[Required]</B><br/>
<samp>int64</samp>
</td>
<td>
   Weight of the resource.</td>
</tr>
    
  
</tbody>
</table>
    


## `ServiceAffinity`     {#kubescheduler-config-k8s-io-v1-ServiceAffinity}
    



**Appears in:**

- [PredicateArgument](#kubescheduler-config-k8s-io-v1-PredicateArgument)

- [PredicateArgument](#kubescheduler-config-k8s-io-v1-PredicateArgument)


ServiceAffinity holds the parameters that are used to configure the corresponding predicate in scheduler policy configuration.

<table class="table">
<thead><tr><th width="30%">Field</th><th>Description</th></tr></thead>
<tbody>
    

  
<tr><td><samp>labels</samp> <B>[Required]</B><br/>
<samp>[]string</samp>
</td>
<td>
   The list of labels that identify node "groups"
All of the labels should match for the node to be considered a fit for hosting the pod</td>
</tr>
    
  
</tbody>
</table>
    


## `ServiceAntiAffinity`     {#kubescheduler-config-k8s-io-v1-ServiceAntiAffinity}
    



**Appears in:**

- [PriorityArgument](#kubescheduler-config-k8s-io-v1-PriorityArgument)

- [PriorityArgument](#kubescheduler-config-k8s-io-v1-PriorityArgument)


ServiceAntiAffinity holds the parameters that are used to configure the corresponding priority function

<table class="table">
<thead><tr><th width="30%">Field</th><th>Description</th></tr></thead>
<tbody>
    

  
<tr><td><samp>label</samp> <B>[Required]</B><br/>
<samp>string</samp>
</td>
<td>
   Used to identify node "groups"</td>
</tr>
    
  
</tbody>
</table>
    


## `UtilizationShapePoint`     {#kubescheduler-config-k8s-io-v1-UtilizationShapePoint}
    



**Appears in:**

- [RequestedToCapacityRatioArguments](#kubescheduler-config-k8s-io-v1-RequestedToCapacityRatioArguments)

- [RequestedToCapacityRatioArguments](#kubescheduler-config-k8s-io-v1-RequestedToCapacityRatioArguments)


UtilizationShapePoint represents single point of priority function shape.

<table class="table">
<thead><tr><th width="30%">Field</th><th>Description</th></tr></thead>
<tbody>
    

  
<tr><td><samp>utilization</samp> <B>[Required]</B><br/>
<samp>int32</samp>
</td>
<td>
   Utilization (x axis). Valid values are 0 to 100. Fully utilized node maps to 100.</td>
</tr>
    
  
<tr><td><samp>score</samp> <B>[Required]</B><br/>
<samp>int32</samp>
</td>
<td>
   Score assigned to given utilization (y axis). Valid values are 0 to 10.</td>
</tr>
    
  
</tbody>
</table>
    
  
