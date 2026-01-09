<h1>Execution Phases – AI Veo3 Video Ad Generation Platform</h1>

<p>
This document describes, in a precise and system-engineering manner, how the platform
executes a single user request from initial input through to final cinematic video delivery.
Each phase corresponds to a functional layer inside the n8n orchestration engine and the
integrated AI services.
</p>

<hr/>

<h2>Phase 1 – User Input Acquisition</h2>

<h3>Objective</h3>
<p>Capture a user’s advertising idea via Telegram in either voice or text form.</p>

<h3>Execution Steps</h3>
<ol>
<li>The Telegram Trigger subscribes to new messages sent to the bot.</li>
<li>The incoming message is parsed for metadata:
<ul>
<li>Chat ID</li>
<li>Message type</li>
<li>Attached media (voice, photo, or none)</li>
</ul>
</li>
<li>An IF-gateway inspects the message payload to determine whether a voice note is present.</li>
</ol>

<table>
<tr><th>Input Type</th><th>Routing Action</th></tr>
<tr><td>Voice Note</td><td>Download file from Telegram API</td></tr>
<tr><td>Text Message</td><td>Pass directly to creative intelligence layer</td></tr>
</table>

<hr/>

<h2>Phase 2 – Speech-to-Text Normalization</h2>

<h3>Objective</h3>
<p>Convert spoken user input into structured text.</p>

<h3>Execution Steps</h3>
<ol>
<li>Telegram file node retrieves the voice file using file_id.</li>
<li>The binary audio payload is forwarded to OpenAI Whisper.</li>
<li>Whisper returns a normalized transcription.</li>
<li>The transcription replaces the original input as the unified ad request.</li>
</ol>

<p>
This phase ensures that all downstream AI components operate on a consistent text-based
representation of the user’s intent.
</p>

<hr/>

<h2>Phase 3 – Creative Intelligence & Script Engineering</h2>

<h3>Objective</h3>
<p>Transform a vague advertising idea into a precise, machine-executable creative plan.</p>

<h3>Execution Steps</h3>
<ol>
<li>The user message is sent to GPT-4o under the “AdScript AI” system persona.</li>
<li>GPT performs internal strategy modeling:
<ul>
<li>Product identification</li>
<li>Target audience definition</li>
<li>Tone and emotional framing</li>
<li>Six-step narrative arc construction</li>
</ul>
</li>
<li>GPT produces a single structured JSON containing:
<ul>
<li>Ad campaign name</li>
<li>Creative summary</li>
<li>Six scene objects with Veo3-optimized prompts</li>
</ul>
</li>
</ol>

<table>
<tr><th>Output Artifact</th><th>Purpose</th></tr>
<tr><td>ad_concept_name</td><td>Unique campaign identifier</td></tr>
<tr><td>creative_summary</td><td>High-level narrative intent</td></tr>
<tr><td>scenes[1–6]</td><td>Individual 5-second cinematic instructions</td></tr>
</table>

<hr/>

<h2>Phase 4 – Parallel Scene Rendering</h2>

<h3>Objective</h3>
<p>Generate six independent cinematic video clips in parallel.</p>

<h3>Execution Model</h3>
<p>
Each scene runs in its own asynchronous execution lane to maximize throughput and fault isolation.
</p>

<table>
<tr><th>Stage</th><th>Description</th></tr>
<tr><td>Submit</td><td>Prompt sent to Veo3 via fal.run</td></tr>
<tr><td>Queue</td><td>Veo3 returns a response_url for polling</td></tr>
<tr><td>Wait</td><td>n8n pauses for model rendering</td></tr>
<tr><td>Poll</td><td>Status checked until video is ready</td></tr>
<tr><td>Fetch</td><td>Video URL returned</td></tr>
<tr><td>Store</td><td>Uploaded to Google Drive</td></tr>
<tr><td>Record</td><td>Airtable updated with scene link</td></tr>
</table>

<hr/>

<h2>Phase 5 – Asset Consolidation & Metadata Synchronization</h2>

<h3>Objective</h3>
<p>Maintain a single source of truth for all generated media.</p>

<h3>Execution Steps</h3>
<ol>
<li>Each scene upload generates a Google Drive webContentLink.</li>
<li>The corresponding Airtable row is created or updated.</li>
<li>Each scene is written into its designated column (scene_link1 … scene_link6).</li>
<li>Upsert logic guarantees idempotency and prevents duplicates.</li>
</ol>

<hr/>

<h2>Phase 6 – Final Video Assembly</h2>

<h3>Objective</h3>
<p>Convert six independent scenes into one cinematic advertisement.</p>

<h3>Execution Steps</h3>
<ol>
<li>Airtable is queried to retrieve all six scene URLs.</li>
<li>json2video API is invoked with:
<ul>
<li>Full-HD resolution</li>
<li>High-quality encoding</li>
<li>Pan, zoom, and fade transitions</li>
</ul>
</li>
<li>The API produces a single merged movie.</li>
<li>The final movie URL is returned to n8n.</li>
</ol>

<hr/>

<h2>Phase 7 – User Delivery</h2>

<h3>Objective</h3>
<p>Return the completed advertisement to the original requester.</p>

<h3>Execution Steps</h3>
<ol>
<li>The final MP4 file is downloaded from json2video.</li>
<li>The file is sent via Telegram’s sendVideo endpoint.</li>
<li>The public movie URL is also sent for sharing and embedding.</li>
</ol>

<hr/>

<h2>Phase 8 – Operational Characteristics</h2>

<table>
<tr><th>Characteristic</th><th>Implementation</th></tr>
<tr><td>Concurrency</td><td>Six parallel rendering pipelines</td></tr>
<tr><td>Reliability</td><td>Polling and waits prevent premature failure</td></tr>
<tr><td>Traceability</td><td>Airtable tracks every asset</td></tr>
<tr><td>Scalability</td><td>Architecture supports thousands of simultaneous ads</td></tr>
</table>

<hr/>

<h2>Conceptual Execution Flow</h2>
<p>
Telegram → Speech AI → Creative AI → Six Parallel Veo3 Jobs → Google Drive + Airtable → json2video → Telegram
</p>

<p>
This phased execution model transforms a single user idea into a fully produced cinematic
advertisement through deterministic, automated AI orchestration.
</p>

</body>
</html>
