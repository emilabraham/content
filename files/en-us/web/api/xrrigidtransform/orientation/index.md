---
title: XRRigidTransform.orientation
slug: Web/API/XRRigidTransform/orientation
tags:
- API
- AR
- Orientation
- Property
- Read-only
- Reality
- Reference
- VR
- Virtual
- WebXR
- WebXR API
- WebXR Device API
- XR
- XRRigidTransform
- augmented
- rotation
browser-compat: api.XRRigidTransform.orientation
---
<p>{{APIRef("WebXR Device API")}}</p>

<p>The read-only {{domxref("XRRigidTransform")}} property
    <code><strong>orientation</strong></code> is a {{domxref("DOMPointReadOnly")}}
    containing a normalized {{Glossary("quaternion")}} (also called a <strong>unit
      quaternion</strong> or <strong>{{interwiki("wikipedia", "versor")}}</strong>)
    specifying the rotational component of the transform represented by the object.
  If you specify a quaternion whose length is not exactly 1.0 meters, it will be
  normalized for you.</p>

<h2 id="Syntax">Syntax</h2>

<pre
  class="brush: js">let <em>orientation</em> = <em>xrRigidTransform</em>.orientation;</pre>

<h3 id="Value">Value</h3>

<p>A {{domxref("DOMPointReadOnly")}} object which contains a unit quaternion providing the
  orientation component of the transform. As a unit quaternion, the length of the returned
  quaternion is always 1.0 meters.</p>

<h2 id="Examples">Examples</h2>

<p>To create a reference space which is oriented to look straight up, positioned 2 meters
  off of ground level:</p>

<pre class="brush: js">xrReferenceSpace = refSpace.getOffsetReferenceSpace(
  new XRRigidTransform({y: -2}, {x: 0.0, y: 1.0, z: 0.0, w: 1.0});
);
</pre>

<p>The unit quaternion specified here is [0.0, 1.0, 0.0, 1.0] to indicate that the object
  should be facing directly along the <em>y</em> axis.</p>

<h2 id="Specifications">Specifications</h2>

{{Specifications}}

<h2 id="Browser_compatibility">Browser compatibility</h2>

<div>{{Compat}}</div>

<h2 id="See_also">See also</h2>

<ul>
  <li><a href="/en-US/docs/Web/API/WebXR_Device_API/Movement_and_motion">Movement,
      orientation, and motion</a></li>
  <li>{{interwiki("wikipedia", "versor", "Unit quaternions")}}</li>
  <li>{{interwiki("wikipedia", "Quaternions and spatial rotation")}}</li>
</ul>