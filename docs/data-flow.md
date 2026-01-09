<h1>Data Flow Architecture – AI Veo3 Video Ad Generation Platform</h1>

<p>
This document defines how data moves through the platform from initial user input to
final video delivery. It specifies data formats, transformation stages, storage layers,
and control flow across all integrated services.
</p>

<hr/>

<h2>1. Data Domains</h2>

<table>
<tr><th>Domain</th><th>Description</th></tr>
<tr><td>User Input</td><td>Telegram text messages and voice recordings</td></tr>
<tr><td>Creative Data</td><td>Transcriptions, scripts, and scene definitions</td></tr>
<tr><td>Media Assets</td><td>AI-generated video files</td></tr>
<tr><td>Metadata</td><td>Campaign identifiers and asset URLs</td></tr>
<tr><td>Final Output</td><td>Merged MP4 advertisement and delivery URLs</td></tr>
</table>

<hr/>

<h2>2. Input Data Flow</h2>

<h3>2.1 Telegram Message Intake</h3>
<ol>
<li>User submits a message to the Telegram bot.</li>
<li>The Telegram Trigger node receives a JSON payload containing:
<ul>
<li>chat.id</li>
<li>message.text or message.voice</li>
<li>file_id if voice is present</li>
</ul>
</li>
<li>This payload becomes the root data object for the workflow.</li>
</ol>

<hr/>

<h2>3. Speech-to-Text Data Transformation</h2>

<h3>3.1 Voice Processing</h3>
<ol>
<li>If a voice note exists, Telegram API returns a binary audio file.</li>
<li>The file is passed to OpenAI Whisper.</li>
<li>Whisper outputs a UTF-8 encoded transcription string.</li>
<li>The original message payload is augmented with a new “text” field.</li>
</ol>

<table>
<tr><th>Input</th><th>Output</th></tr>
<tr><td>Audio binary</td><td>Normalized transcription string</td></tr>
</table>

<hr/>

<h2>4. Creative Data Flow</h2>

<h3>4.1 Prompt Engineering and Script Generation</h3>
<ol>
<li>The transcription or original text is sent to GPT-4o.</li>
<li>GPT returns a structured JSON object containing:
<ul>
<li>Campaign name</li>
<li>Creative summary</li>
<li>Six scene definitions</li>
</ul>
</li>
<li>This JSON becomes the authoritative creative blueprint for the entire workflow.</li>
</ol>

<table>
<tr><th>Field</th><th>Purpose</th></tr>
<tr><td>ad_concept_name</td><td>Campaign identity</td></tr>
<tr><td>creative_summary</td><td>Creative strategy</td></tr>
<tr><td>scenes[].veo_prompt</td><td>Executable video instructions</td></tr>
</table>

<hr/>

<h2>5. Video Generation Data Flow</h2>

<h3>5.1 Scene Dispatch</h3>
<ol>
<li>Each scene’s prompt is sent to Veo3 via fal.run.</li>
<li>The API returns a response_url for asynchronous polling.</li>
<li>These URLs are stored in the n8n execution context.</li>
</ol>

<h3>5.2 Rendering Completion</h3>
<ol>
<li>n8n polls each response_url.</li>
<li>When completed, Veo3 returns a JSON containing:
<ul>
<li>video.url</li>
<li>job status</li>
</ul>
</li>
</ol>

<hr/>

<h2>6. Asset Storage Data Flow</h2>

<h3>6.1 Google Drive Upload</h3>
<ol>
<li>The video.url is fetched.</li>
<li>The MP4 file is uploaded to Google Drive.</li>
<li>A webContentLink is returned.</li>
</ol>

<h3>6.2 Airtable Synchronization</h3>
<ol>
<li>The campaign record is created or updated.</li>
<li>Each scene link is written to its designated column.</li>
<li>This forms the system-of-record for all generated assets.</li>
</ol>

<table>
<tr><th>Storage</th><th>Data Stored</th></tr>
<tr><td>Google Drive</td><td>Binary video files</td></tr>
<tr><td>Airtable</td><td>URLs and campaign metadata</td></tr>
</table>

<hr/>

<h2>7. Final Video Data Flow</h2>

<h3>7.1 Retrieval</h3>
<ol>
<li>Airtable is queried for all six scene URLs.</li>
<li>The URLs are passed to json2video.</li>
</ol>

<h3>7.2 Assembly</h3>
<ol>
<li>json2video merges scenes with transitions.</li>
<li>A final MP4 and movie URL are produced.</li>
</ol>

<hr/>

<h2>8. Delivery Data Flow</h2>

<ol>
<li>The final MP4 is downloaded.</li>
<li>The file is sent to Telegram.</li>
<li>The public URL is sent as a message.</li>
</ol>

<hr/>

<h2>9. End-to-End Data Flow Diagram</h2>

<p>
Telegram Input → Transcription → Creative JSON → Six Veo3 Jobs → Google Drive Files → Airtable Links → json2video → Final MP4 → Telegram Delivery
</p>

<hr/>

<h2>10. Data Integrity & Governance</h2>

<table>
<tr><th>Control</th><th>Implementation</th></tr>
<tr><td>Idempotency</td><td>Airtable upsert by campaign name</td></tr>
<tr><td>Traceability</td><td>Each scene URL stored and retrievable</td></tr>
<tr><td>Consistency</td><td>Single creative JSON drives all steps</td></tr>
<tr><td>Auditability</td><td>Drive + Airtable form immutable history</td></tr>
</table>

</body>
</html>
