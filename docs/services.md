<h1>Services Architecture – AI Veo3 Video Ad Generation Platform</h1>

<p>
This document provides a comprehensive, service-level architectural breakdown of the
AI Veo3 Video Ad Generation Platform. Each service is described in terms of its role,
responsibilities, data contracts, execution boundaries, and interactions with other services.
The platform follows a modular, loosely coupled service model orchestrated centrally by n8n.
</p>

<hr/>

<h2>1. Service-Oriented Architecture Overview</h2>

<p>
The platform is designed as a logical Service-Oriented Architecture (SOA) where each external
integration or AI capability is treated as a discrete service. n8n acts as the orchestration
and control plane, coordinating data flow and execution order.
</p>

<table>
<tr>
<th>Service Category</th>
<th>Primary Responsibility</th>
</tr>
<tr>
<td>Ingress Services</td>
<td>User interaction and request intake</td>
</tr>
<tr>
<td>Intelligence Services</td>
<td>Speech understanding and creative reasoning</td>
</tr>
<tr>
<td>Generation Services</td>
<td>Video asset creation</td>
</tr>
<tr>
<td>Persistence Services</td>
<td>Asset and metadata storage</td>
</tr>
<tr>
<td>Composition Services</td>
<td>Final video assembly</td>
</tr>
<tr>
<td>Egress Services</td>
<td>Delivery and notification</td>
</tr>
</table>

<hr/>

<h2>2. Ingress Services</h2>

<h3>2.1 Telegram Messaging Service</h3>

<h4>Purpose</h4>
<p>
Acts as the primary user-facing interface for submitting ad generation requests and receiving
final outputs.
</p>

<h4>Responsibilities</h4>
<ul>
<li>Accept text messages describing ad concepts</li>
<li>Accept voice notes containing spoken ad ideas</li>
<li>Transmit chat metadata for response routing</li>
<li>Deliver final video files and URLs to users</li>
</ul>

<h4>Data Characteristics</h4>
<table>
<tr><th>Data Element</th><th>Description</th></tr>
<tr><td>chat.id</td><td>Unique conversation identifier</td></tr>
<tr><td>message.text</td><td>User-provided textual input</td></tr>
<tr><td>message.voice.file_id</td><td>Reference to voice audio stored by Telegram</td></tr>
</table>

<hr/>

<h2>3. Orchestration Service</h2>

<h3>3.1 n8n Workflow Orchestrator</h3>

<h4>Purpose</h4>
<p>
Serves as the central execution engine that coordinates all services, enforces sequencing,
manages branching, and handles asynchronous operations.
</p>

<h4>Responsibilities</h4>
<ul>
<li>Event-driven workflow initiation</li>
<li>Conditional routing based on input type</li>
<li>Parallel execution of scene generation pipelines</li>
<li>Retry, wait, and polling logic for async services</li>
<li>Credential isolation and secure service access</li>
</ul>

<h4>Architectural Role</h4>
<p>
n8n functions as a control plane rather than a data store, maintaining transient execution
state and passing structured data between services.
</p>

<hr/>

<h2>4. Intelligence Services</h2>

<h3>4.1 Speech Recognition Service (OpenAI Whisper)</h3>

<h4>Purpose</h4>
<p>
Converts unstructured spoken audio into normalized text suitable for downstream AI reasoning.
</p>

<h4>Responsibilities</h4>
<ul>
<li>Audio decoding</li>
<li>Language detection</li>
<li>High-accuracy transcription</li>
</ul>

<h4>Input / Output</h4>
<table>
<tr><th>Input</th><th>Output</th></tr>
<tr><td>Binary audio stream</td><td>UTF-8 encoded transcription string</td></tr>
</table>

<hr/>

<h3>4.2 Creative Reasoning Service (GPT-4o)</h3>

<h4>Purpose</h4>
<p>
Acts as an autonomous creative director that transforms user intent into a structured,
machine-executable advertising plan.
</p>

<h4>Responsibilities</h4>
<ul>
<li>Semantic understanding of user intent</li>
<li>Narrative arc construction</li>
<li>Prompt engineering optimized for Veo3</li>
<li>Strict schema-compliant JSON generation</li>
</ul>

<h4>Output Contract</h4>
<table>
<tr><th>Field</th><th>Description</th></tr>
<tr><td>ad_concept_name</td><td>Campaign identifier</td></tr>
<tr><td>creative_summary</td><td>One-line creative strategy</td></tr>
<tr><td>scenes[]</td><td>Six scene definitions with cinematic prompts</td></tr>
</table>

<hr/>

<h2>5. Generation Services</h2>

<h3>5.1 Video Generation Service (Veo3 via fal.run)</h3>

<h4>Purpose</h4>
<p>
Generates short-form cinematic video clips from natural language prompts.
</p>

<h4>Responsibilities</h4>
<ul>
<li>Accept scene-specific prompts</li>
<li>Render photorealistic video with audio</li>
<li>Operate asynchronously via job queue</li>
</ul>

<h4>Execution Characteristics</h4>
<table>
<tr><th>Characteristic</th><th>Description</th></tr>
<tr><td>Invocation</td><td>HTTP POST per scene</td></tr>
<tr><td>Processing</td><td>Asynchronous job queue</td></tr>
<tr><td>Result</td><td>Downloadable video URL</td></tr>
</table>

<hr/>

<h2>6. Persistence Services</h2>

<h3>6.1 Asset Storage Service (Google Drive)</h3>

<h4>Purpose</h4>
<p>
Provides durable, scalable storage for all generated video assets.
</p>

<h4>Responsibilities</h4>
<ul>
<li>Store individual scene videos</li>
<li>Provide public or internal access links</li>
<li>Maintain long-term asset retention</li>
</ul>

<h3>6.2 Metadata Management Service (Airtable)</h3>

<h4>Purpose</h4>
<p>
Acts as the system of record for campaign metadata and asset references.
</p>

<h4>Responsibilities</h4>
<ul>
<li>Create campaign records</li>
<li>Upsert scene links in deterministic fields</li>
<li>Support querying for final assembly</li>
</ul>

<table>
<tr><th>Field Group</th><th>Usage</th></tr>
<tr><td>Name</td><td>Primary campaign key</td></tr>
<tr><td>scene_link1–6</td><td>Scene-level asset URLs</td></tr>
</table>

<hr/>

<h2>7. Composition Services</h2>

<h3>7.1 Video Assembly Service (json2video)</h3>

<h4>Purpose</h4>
<p>
Transforms multiple independent video clips into a single, polished cinematic advertisement.
</p>

<h4>Responsibilities</h4>
<ul>
<li>Sequence scene videos</li>
<li>Apply pan, zoom, and fade effects</li>
<li>Render final Full-HD MP4</li>
</ul>

<h4>Data Dependencies</h4>
<p>
Consumes scene URLs retrieved from Airtable, ensuring deterministic and repeatable assembly.
</p>

<hr/>

<h2>8. Egress Services</h2>

<h3>8.1 Telegram Delivery Service</h3>

<h4>Purpose</h4>
<p>
Delivers the final output back to the user through the same channel used for request submission.
</p>

<h4>Responsibilities</h4>
<ul>
<li>Transmit MP4 video files</li>
<li>Send public URLs for sharing</li>
<li>Ensure correct chat routing</li>
</ul>

<hr/>

<h2>9. Inter-Service Interaction Flow</h2>

<p>
Telegram → n8n → Whisper → GPT-4o → Veo3 (×6) → Google Drive → Airtable → json2video → Telegram
</p>

<hr/>

<h2>10. Service-Level Characteristics</h2>

<table>
<tr><th>Aspect</th><th>Implementation</th></tr>
<tr><td>Loose Coupling</td><td>Services communicate via structured JSON only</td></tr>
<tr><td>Scalability</td><td>Parallel video generation lanes</td></tr>
<tr><td>Fault Isolation</td><td>Per-scene execution boundaries</td></tr>
<tr><td>Observability</td><td>Airtable and Drive provide traceability</td></tr>
<tr><td>Extensibility</td><td>New services can be inserted without redesign</td></tr>
</table>

<hr/>

<h2>11. Architectural Summary</h2>

<p>
The services architecture decomposes the ad generation lifecycle into clearly defined,
independently scalable services coordinated by a central orchestration engine. This design
enables the platform to function as a fully automated AI advertising studio, capable of
operating at production scale with minimal human intervention.
</p>

</body>
</html>
