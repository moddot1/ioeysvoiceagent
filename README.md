// © 2026 IOEYS. All Rights Reserved.
// Jack of All Codes v3.1 - Voice Activated
// heyjack.uiuxutility.workers.dev

const VOICE_HTML = `<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Jack of All Codes</title>
<style>
  * { margin:0; padding:0; box-sizing:border-box; }
  body { font-family: Arial, sans-serif; background: radial-gradient(circle at 10% 20%, #0b0f1c, #03050a); color: white; min-height: 100vh; display: flex; flex-direction: column; align-items: center; justify-content: center; padding: 20px; }
  h1 { font-size: 2.5em; background: linear-gradient(135deg, #c0d0ff, #a0b5f0); -webkit-background-clip: text; -webkit-text-fill-color: transparent; margin-bottom: 10px; text-align: center; }
  .subtitle { color: #a0b0d0; margin-bottom: 40px; text-align: center; }
  .orb { width: 180px; height: 180px; border-radius: 50%; background: radial-gradient(circle, #1e3a8a, #0b0f1c); border: 3px solid #4f7fff; display: flex; align-items: center; justify-content: center; font-size: 4em; cursor: pointer; transition: all 0.3s ease; box-shadow: 0 0 30px rgba(79,127,255,0.3); margin-bottom: 30px; }
  .orb.listening { animation: pulse 1.5s infinite; border-color: #ff4444; box-shadow: 0 0 60px rgba(255,68,68,0.5); }
  .orb.speaking { animation: pulse 0.8s infinite; border-color: #44ff88; box-shadow: 0 0 60px rgba(68,255,136,0.5); }
  @keyframes pulse { 0%, 100% { transform: scale(1); } 50% { transform: scale(1.08); } }
  .status { font-size: 1.3em; margin-bottom: 20px; color: #a0b0d0; text-align: center; min-height: 40px; }
  .transcript { background: rgba(255,255,255,0.05); border: 1px solid #2a3a5a; border-radius: 15px; padding: 15px 20px; width: 100%; max-width: 500px; min-height: 60px; margin-bottom: 15px; font-size: 1em; color: #c0d0ff; text-align: center; }
  .response { background: rgba(79,127,255,0.1); border: 1px solid #4f7fff; border-radius: 15px; padding: 15px 20px; width: 100%; max-width: 500px; min-height: 60px; margin-bottom: 30px; font-size: 1em; color: white; text-align: center; line-height: 1.6; }
  .instructions { color: #4a5a78; font-size: 0.85em; text-align: center; max-width: 400px; line-height: 1.8; }
  .instructions span { color: #4f7fff; font-weight: bold; }
  .btn { background: #1e3a8a; border: 1px solid #4f7fff; color: white; padding: 12px 30px; border-radius: 30px; font-size: 1em; cursor: pointer; margin: 5px; transition: all 0.2s; }
  .btn:hover { background: #2a4f9f; }
  .btn.active { background: #ff4444; border-color: #ff4444; }
  .links { margin-top: 30px; display: flex; flex-wrap: wrap; gap: 10px; justify-content: center; }
  .link-btn { background: rgba(79,127,255,0.1); border: 1px solid #4f7fff; color: #a0b0d0; padding: 8px 16px; border-radius: 20px; font-size: 0.8em; cursor: pointer; text-decoration: none; }
</style>
</head>
<body>
<h1>Jack of All Codes by IOEYS</h1>
<p class="subtitle">Your Voice Activated AI Development Agent</p>
<div class="orb" id="orb" onclick="toggleListening()">MIC</div>
<div class="status" id="status">Tap the mic to activate Jack</div>
<div class="transcript" id="transcript">Your words appear here...</div>
<div class="response" id="response">Jack's response appears here...</div>
<div>
  <button class="btn" id="micBtn" onclick="toggleListening()">Start listening</button>
  <button class="btn" onclick="speakResponse('Jack of All Codes is operational and ready to deploy your code.')">Test voice</button>
</div>
<div class="instructions">
  <p>Say <span>"Jack"</span> to wake me up</p>
  <p>Say <span>"Over"</span> when you finish speaking</p>
  <p>Say <span>"Over and out"</span> to end</p>
  <br>
  <p>Try: <span>"Jack, scan my repos"</span></p>
  <p>Try: <span>"Jack, deploy my project to [project name]"</span></p>
  <p>Try: <span>"Jack, status report"</span></p>
</div>
<div class="links">
  <a class="link-btn" href="/api/status">Status</a>
  <a class="link-btn" href="/api/scan">Scan repos</a>
</div>
<script>
const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
const synth = window.speechSynthesis;
let recognition = null;
let isListening = false;
let conversationActive = false;
let pendingDeployProject = null;

function initRecognition() {
  if (!SpeechRecognition) {
    document.getElementById('status').innerText = 'Voice not supported in this browser. Try Chrome.';
    return false;
  }
  recognition = new SpeechRecognition();
  recognition.continuous = true;
  recognition.interimResults = false;
  recognition.lang = 'en-US';
  recognition.onresult = (event) => {
    const last = event.results.length - 1;
    const transcript = event.results[last][0].transcript.toLowerCase().trim();
    document.getElementById('transcript').innerText = transcript;
    processCommand(transcript);
  };
  recognition.onerror = (e) => {
    document.getElementById('status').innerText = 'Mic error: ' + e.error;
  };
  recognition.onend = () => { if (isListening) recognition.start(); };
  return true;
}

function toggleListening() {
  if (!recognition && !initRecognition()) return;
  if (isListening) {
    recognition.stop();
    isListening = false;
    conversationActive = false;
    document.getElementById('micBtn').innerText = 'Start listening';
    document.getElementById('micBtn').classList.remove('active');
    document.getElementById('orb').classList.remove('listening');
    document.getElementById('status').innerText = 'Tap the mic to activate Jack';
  } else {
    recognition.start();
    isListening = true;
    document.getElementById('micBtn').innerText = 'Stop listening';
    document.getElementById('micBtn').classList.add('active');
    document.getElementById('orb').classList.add('listening');
    document.getElementById('status').innerText = 'Listening... say "Jack" to activate';
  }
}

function processCommand(text) {
  if (text.includes('jack') && !conversationActive) {
    conversationActive = true;
    document.getElementById('status').innerText = 'Jack activated';
    speakResponse("Jack here. What do you need? Over.");
    return;
  }
  if (text.includes('over and out')) {
    conversationActive = false;
    pendingDeployProject = null;
    document.getElementById('status').innerText = 'Standby...';
    speakResponse("Over and out. Standing by.");
    return;
  }
  if (!conversationActive) return;

  if (pendingDeployProject === 'waiting_for_name') {
    const name = text.replace('over', '').trim();
    pendingDeployProject = null;
    speakResponse("Deploying " + name + " now.");
    fetch('/api/deploy', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ project: name })
    })
    .then(r => r.json())
    .then(data => {
      const msg = data.status === 'created_and_deploying' || data.status === 'deploy_triggered'
        ? "Deploy triggered for " + name + ". Over."
        : "Deploy failed. " + (data.error || "Check the project name.") + " Over.";
      speakResponse(msg);
    })
    .catch(() => speakResponse("Deploy request failed. Over."));
    return;
  }

  if (text.includes('scan') || text.includes('repos') || text.includes('github')) {
    speakResponse("Scanning your GitHub repositories now.");
    fetch('/api/scan')
      .then(r => r.json())
      .then(data => {
        const msg = data.total
          ? "Found " + data.total + " repositories. Over."
          : "GitHub token needed. Check your settings. Over.";
        speakResponse(msg);
      })
      .catch(() => speakResponse("Scan error. Check GitHub token. Over."));
  } else if (text.includes('deploy')) {
    pendingDeployProject = 'waiting_for_name';
    speakResponse("Which project? Say the name then over.");
  } else if (text.includes('status')) {
    fetch('/api/status')
      .then(r => r.json())
      .then(data => {
        speakResponse("Jack is operational. GitHub " + data.github + ". Cloudflare " + data.cloudflare + ". Over.");
      });
  } else if (text.includes('over')) {
    speakResponse("Over. Listening.");
  } else {
    speakResponse("I can scan repos, deploy projects, or check status. What do you need? Over.");
  }
}

function speakResponse(text) {
  document.getElementById('response').innerText = text;
  document.getElementById('orb').classList.remove('listening');
  document.getElementById('orb').classList.add('speaking');
  const utterance = new SpeechSynthesisUtterance(text);
  utterance.rate = 1.0;
  utterance.pitch = 0.9;
  utterance.volume = 1.0;
  utterance.onend = () => {
    document.getElementById('orb').classList.remove('speaking');
    if (isListening) document.getElementById('orb').classList.add('listening');
  };
  synth.cancel();
  synth.speak(utterance);
}

setTimeout(() => { if (initRecognition()) toggleListening(); }, 2000);
</script>
</body>
</html>`;

function checkAllowedIP(request, env) {
  if (!env.ALLOWED_IP) return true;
  const ip = request.headers.get('CF-Connecting-IP');
  return ip === env.ALLOWED_IP;
}

export default {
  async fetch(request, env) {
    const url = new URL(request.url);
    const path = url.pathname;

    const headers = {
      'Content-Type': 'application/json',
      'Access-Control-Allow-Origin': '*',
      'X-Frame-Options': 'DENY',
      'X-Content-Type-Options': 'nosniff',
      'Strict-Transport-Security': 'max-age=31536000'
    };

    if (path === '/' || path === '/voice') {
      return new Response(VOICE_HTML, { headers: { 'Content-Type': 'text/html' } });
    }

    if (path === '/health') {
      return new Response(JSON.stringify({
        name: 'Jack of All Codes', version: '3.1.0', status: 'operational',
        voice: 'enabled', timestamp: new Date().toISOString()
      }, null, 2), { headers });
    }

    if (path === '/api/scan') {
      const githubToken = env.GITHUB_TOKEN;
      if (!githubToken) {
        return new Response(JSON.stringify({ error: 'GitHub token not configured' }), { status: 400, headers });
      }
      const response = await fetch('https://api.github.com/user/repos?sort=updated&per_page=100', {
        headers: { 'Authorization': `token ${githubToken}`, 'User-Agent': 'Jack-of-All-Codes' }
      });
      const repos = await response.json();
      return new Response(JSON.stringify({
        status: 'success', total: repos.length,
        repositories: repos.map(r => ({ name: r.name, language: r.language, updated: r.updated_at, url: r.html_url, private: r.private }))
      }, null, 2), { headers });
    }

    if (path === '/api/status') {
      return new Response(JSON.stringify({
        name: 'Jack of All Codes', version: '3.1.0', status: 'operational', voice: 'enabled',
        github: env.GITHUB_TOKEN ? 'connected' : 'not configured',
        cloudflare: env.CF_API_TOKEN ? 'connected' : 'not configured',
        ip_restriction_enforced: !!env.ALLOWED_IP,
        timestamp: new Date().toISOString()
      }, null, 2), { headers });
    }

    if (path === '/api/deploy' && request.method === 'POST') {
      if (!checkAllowedIP(request, env)) {
        return new Response(JSON.stringify({ error: 'forbidden' }), { status: 403, headers });
      }
      const body = await request.json();
      const projectName = body.project;
      if (!projectName) {
        return new Response(JSON.stringify({ error: 'Project name required' }), { status: 400, headers });
      }

      const checkRes = await fetch(
        `https://api.cloudflare.com/client/v4/accounts/${env.CF_ACCOUNT_ID}/pages/projects/${projectName}`,
        { headers: { Authorization: `Bearer ${env.CF_API_TOKEN}` } }
      );

      if (!checkRes.ok) {
        const createRes = await fetch(
          `https://api.cloudflare.com/client/v4/accounts/${env.CF_ACCOUNT_ID}/pages/projects`,
          {
            method: 'POST',
            headers: { Authorization: `Bearer ${env.CF_API_TOKEN}`, 'Content-Type': 'application/json' },
            body: JSON.stringify({ name: projectName, production_branch: 'main' })
          }
        );
        if (!createRes.ok) {
          const err = await createRes.text();
          return new Response(JSON.stringify({ error: 'Pages create failed', detail: err }), { status: 502, headers });
        }
        const created = await createRes.json();
        return new Response(JSON.stringify({ status: 'created_and_deploying', project: created.result }), { headers });
      }

      const deployRes = await fetch(
        `https://api.cloudflare.com/client/v4/accounts/${env.CF_ACCOUNT_ID}/pages/projects/${projectName}/deployments`,
        { method: 'POST', headers: { Authorization: `Bearer ${env.CF_API_TOKEN}` } }
      );
      if (!deployRes.ok) {
        const err = await deployRes.text();
        return new Response(JSON.stringify({ error: 'Deploy trigger failed', detail: err }), { status: 502, headers });
      }
      const deployment = await deployRes.json();
      return new Response(JSON.stringify({ status: 'deploy_triggered', deployment: deployment.result }), { headers });
    }

    return new Response(JSON.stringify({ error: 'Route not found' }), { status: 404, headers });
  }
};
