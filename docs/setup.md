<h1>System Setup – AI Veo3 Video Ad Generation Platform</h1>

<p>
This document defines the complete enterprise-grade setup process required to deploy,
configure, and operate the AI Veo3 Video Ad Generation Platform in a production environment.
It covers infrastructure, credentials, service provisioning, workflow import, and validation.
</p>

<hr/>

<h2>1. Infrastructure Prerequisites</h2>

<table>
<tr><th>Component</th><th>Requirement</th></tr>
<tr><td>n8n Server</td><td>Docker, VPS, or managed n8n instance</td></tr>
<tr><td>Storage</td><td>Google Drive with dedicated folder</td></tr>
<tr><td>Database</td><td>Airtable base with predefined schema</td></tr>
<tr><td>Network</td><td>Public internet access with HTTPS</td></tr>
<tr><td>Security</td><td>Environment variable isolation</td></tr>
</table>

<hr/>

<h2>2. External Service Provisioning</h2>

<h3>2.1 Telegram Bot</h3>
<ol>
<li>Create a Telegram bot using BotFather.</li>
<li>Obtain the Bot Token.</li>
<li>Configure webhook or polling mode in n8n.</li>
</ol>

<h3>2.2 OpenAI</h3>
<ol>
<li>Create an OpenAI account.</li>
<li>Generate an API key.</li>
<li>Ensure access to:
<ul>
<li>Whisper (speech-to-text)</li>
<li>GPT-4o (creative generation)</li>
</ul>
</li>
</ol>

<h3>2.3 Veo3 via fal.run</h3>
<ol>
<li>Register on fal.run.</li>
<li>Enable Veo3 model access.</li>
<li>Generate API Key and Secret.</li>
</ol>

<h3>2.4 Google Drive</h3>
<ol>
<li>Create a dedicated folder for generated videos.</li>
<li>Enable OAuth credentials.</li>
<li>Grant write access to n8n.</li>
</ol>

<h3>2.5 Airtable</h3>
<ol>
<li>Create a new Base.</li>
<li>Create a Table with fields:
<ul>
<li>Name</li>
<li>voice_over</li>
<li>scene_link1 through scene_link6</li>
</ul>
</li>
<li>Generate a Personal Access Token.</li>
</ol>

<h3>2.6 json2video</h3>
<ol>
<li>Create a json2video account.</li>
<li>Generate an API key.</li>
<li>Enable Full-HD video rendering.</li>
</ol>

<hr/>

<h2>3. Credential Management</h2>

<table>
<tr><th>Variable</th><th>Source</th></tr>
<tr><td>TELEGRAM_BOT_TOKEN</td><td>Telegram BotFather</td></tr>
<tr><td>OPENAI_API_KEY</td><td>OpenAI dashboard</td></tr>
<tr><td>VEO3_API_KEY</td><td>fal.run portal</td></tr>
<tr><td>GOOGLE_DRIVE_OAUTH</td><td>Google Cloud Console</td></tr>
<tr><td>AIRTABLE_TOKEN</td><td>Airtable developer hub</td></tr>
<tr><td>JSON2VIDEO_API_KEY</td><td>json2video dashboard</td></tr>
</table>

<p>
All credentials must be stored in n8n’s secure credential store or environment variables.
</p>

<hr/>

<h2>4. Workflow Deployment</h2>

<h3>4.1 Importing the Workflow</h3>
<ol>
<li>Open n8n Editor UI.</li>
<li>Select “Import Workflow”.</li>
<li>Upload the Veo3 Ad Generator JSON file.</li>
<li>Validate all nodes load successfully.</li>
</ol>

<h3>4.2 Credential Binding</h3>
<ol>
<li>Assign Telegram credentials to all Telegram nodes.</li>
<li>Assign OpenAI credentials to Whisper and GPT nodes.</li>
<li>Assign fal.run credentials to all Veo3 HTTP nodes.</li>
<li>Assign Google Drive OAuth to Drive nodes.</li>
<li>Assign Airtable token to all Airtable nodes.</li>
<li>Assign json2video API key to video assembly nodes.</li>
</ol>

<hr/>

<h2>5. Environment Validation</h2>

<table>
<tr><th>Validation Step</th><th>Expected Result</th></tr>
<tr><td>Telegram test message</td><td>Workflow triggers</td></tr>
<tr><td>Voice transcription</td><td>Text returned</td></tr>
<tr><td>GPT scene generation</td><td>Six scene JSON</td></tr>
<tr><td>Veo3 rendering</td><td>Video URLs returned</td></tr>
<tr><td>Drive upload</td><td>Files appear in folder</td></tr>
<tr><td>Airtable update</td><td>Links populated</td></tr>
<tr><td>json2video output</td><td>Final MP4 created</td></tr>
<tr><td>Telegram delivery</td><td>User receives video</td></tr>
</table>

<hr/>

<h2>6. Deployment Topology</h2>

<p>
Telegram Clients → Telegram API → n8n Server → AI & Media Services → Cloud Storage → Telegram
</p>

<hr/>

<h2>7. Production Readiness Checklist</h2>

<ul>
<li>HTTPS enabled on n8n</li>
<li>Rate limits configured</li>
<li>API keys rotated and secured</li>
<li>Google Drive storage monitored</li>
<li>Airtable capacity planned</li>
<li>Veo3 quota verified</li>
</ul>

<hr/>

<h2>8. Operational Notes</h2>

<p>
The system is designed for continuous operation. New ads can be submitted at any time and
are processed independently. Parallel rendering ensures that large volumes of requests
can be handled without blocking or resource contention.
</p>

</body>
</html>
