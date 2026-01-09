<h1>AI Veo3 Video Ad Generation Platform – System Architecture</h1>

<h2>1. Architectural Overview</h2>
<p>
This platform is an enterprise-grade AI orchestration system designed to transform a user’s
voice or text-based advertisement idea into a fully produced, multi-scene cinematic video
commercial. The entire pipeline is implemented as a distributed, asynchronous workflow in
n8n and integrates multiple AI and cloud services.
</p>

<h3>Primary Architecture Pattern</h3>
<ul>
<li>Event-Driven Orchestration</li>
<li>Asynchronous Job Processing</li>
<li>Parallel Scene Rendering</li>
<li>Centralized Asset & Metadata Management</li>
</ul>

<hr/>

<h2>2. System-Level Architecture</h2>

<table>
<tr><th>Layer</th><th>Technology</th><th>Purpose</th></tr>
<tr><td>User Interface</td><td>Telegram Bot</td><td>Collects voice or text ad requests and delivers final video</td></tr>
<tr><td>Orchestration</td><td>n8n</td><td>Controls workflow execution, branching, waiting, and retries</td></tr>
<tr><td>Speech AI</td><td>OpenAI Whisper</td><td>Converts Telegram voice notes into text</td></tr>
<tr><td>Creative Intelligence</td><td>GPT-4o</td><td>Generates structured six-scene ad scripts</td></tr>
<tr><td>Video Generation</td><td>Veo 3 (via fal.run)</td><td>Creates cinematic videos from scene prompts</td></tr>
<tr><td>Asset Storage</td><td>Google Drive</td><td>Stores all generated scene videos</td></tr>
<tr><td>Metadata Store</td><td>Airtable</td><td>Tracks campaigns and scene URLs</td></tr>
<tr><td>Video Assembly</td><td>json2video</td><td>Merges six scenes into a single final movie</td></tr>
<tr><td>Delivery</td><td>Telegram API</td><td>Sends MP4 and URL back to user</td></tr>
</table>

<hr/>

<h2>3. End-to-End Execution Flow</h2>

<h3>3.1 Input Capture</h3>
<ol>
<li>User sends either a voice note or text message to the Telegram bot.</li>
<li>Telegram Trigger node activates the n8n workflow.</li>
<li>An IF gateway checks if the message contains a voice attachment.</li>
<li>If voice is present, the file is downloaded via Telegram API.</li>
<li>The audio is transcribed using OpenAI Whisper.</li>
<li>The resulting text becomes the unified ad request.</li>
</ol>

<h3>3.2 Creative Intelligence Processing</h3>
<ol>
<li>The transcribed or typed user message is sent to GPT-4o.</li>
<li>GPT executes an internal strategy:
<ul>
<li>Identifies product, audience, tone, and core message</li>
<li>Builds a six-step narrative arc</li>
<li>Generates six Veo-optimized cinematic prompts</li>
</ul>
</li>
<li>GPT returns a structured JSON containing:
<ul>
<li>Ad concept name</li>
<li>Creative summary</li>
<li>Six scene objects with titles and Veo prompts</li>
</ul>
</li>
</ol>

<hr/>

<h2>4. Parallel Scene Rendering Architecture</h2>

<p>
Each of the six scenes is processed through an independent asynchronous rendering lane.
This allows all videos to be generated in parallel for maximum throughput.
</p>

<table>
<tr><th>Stage</th><th>Description</th></tr>
<tr><td>Submit</td><td>POST request sent to fal.run Veo3 endpoint with scene prompt</td></tr>
<tr><td>Queue</td><td>Veo3 returns a response_url for polling</td></tr>
<tr><td>Wait</td><td>n8n pauses execution for several minutes</td></tr>
<tr><td>Poll</td><td>System checks Veo3 job status via response_url</td></tr>
<tr><td>Download</td><td>Video URL retrieved once rendering is complete</td></tr>
<tr><td>Store</td><td>Video file uploaded to Google Drive</td></tr>
<tr><td>Record</td><td>Airtable updated with the scene link</td></tr>
</table>

<hr/>

<h2>5. Centralized Asset & Metadata Model</h2>

<h3>Airtable Campaign Schema</h3>

<table>
<tr><th>Field</th><th>Description</th></tr>
<tr><td>Name</td><td>Ad campaign identifier</td></tr>
<tr><td>scene_link1 – scene_link6</td><td>Google Drive URLs for each scene</td></tr>
<tr><td>voice_over</td><td>Original transcription (optional)</td></tr>
</table>

<p>
This schema allows:
</p>
<ul>
<li>Tracking of all generated assets</li>
<li>Idempotent updates from parallel branches</li>
<li>Reliable retrieval for final video assembly</li>
</ul>

<hr/>

<h2>6. Final Movie Assembly</h2>

<ol>
<li>Airtable is queried to retrieve all six scene URLs.</li>
<li>json2video API is invoked with:
<ul>
<li>Full-HD resolution</li>
<li>Zoom, pan, and fade transitions per scene</li>
</ul>
</li>
<li>The service stitches the six clips into a single cinematic MP4.</li>
<li>A project URL and final movie URL are returned.</li>
</ol>

<hr/>

<h2>7. Delivery & User Notification</h2>

<ol>
<li>The final MP4 is downloaded from json2video.</li>
<li>The video file is sent to the original Telegram chat.</li>
<li>A public URL of the movie is also sent for easy sharing.</li>
</ol>

<hr/>

<h2>8. Architectural Characteristics</h2>

<table>
<tr><th>Property</th><th>Implementation</th></tr>
<tr><td>Scalability</td><td>Parallel scene pipelines allow horizontal scaling</td></tr>
<tr><td>Fault Tolerance</td><td>Each scene is isolated; failures do not corrupt others</td></tr>
<tr><td>Extensibility</td><td>New models, scenes, or delivery channels can be added</td></tr>
<tr><td>Auditability</td><td>Airtable provides full asset traceability</td></tr>
<tr><td>Security</td><td>API keys isolated via environment variables</td></tr>
</table>

<hr/>

<h2>9. Conceptual Architecture Diagram</h2>

<p>
Telegram → n8n → OpenAI → Veo3 → Google Drive → Airtable → json2video → Telegram
</p>

<p>
This represents a complete AI-driven creative production pipeline replacing
scriptwriters, video editors, asset managers, and delivery teams with a fully
automated orchestration engine.
</p>

</body>
</html>
