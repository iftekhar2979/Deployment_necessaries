<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>GitHub Actions Visual Guide</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            padding: 20px;
            color: #333;
        }

        .container {
            max-width: 1200px;
            margin: 0 auto;
        }

        h1 {
            text-align: center;
            color: white;
            margin-bottom: 10px;
            font-size: 2.5em;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
        }

        .subtitle {
            text-align: center;
            color: #f0f0f0;
            margin-bottom: 40px;
            font-size: 1.1em;
        }

        .section {
            background: white;
            border-radius: 12px;
            padding: 30px;
            margin-bottom: 30px;
            box-shadow: 0 8px 32px rgba(0,0,0,0.1);
            animation: slideIn 0.6s ease;
        }

        @keyframes slideIn {
            from {
                opacity: 0;
                transform: translateY(20px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }

        .section h2 {
            color: #667eea;
            margin-bottom: 20px;
            font-size: 1.8em;
            border-bottom: 3px solid #667eea;
            padding-bottom: 10px;
        }

        /* Color coded boxes */
        .concept-box {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
            gap: 20px;
            margin-top: 20px;
        }

        .concept {
            padding: 20px;
            border-radius: 10px;
            color: white;
            transition: transform 0.3s ease;
        }

        .concept:hover {
            transform: translateY(-5px);
        }

        .concept h3 {
            margin-bottom: 10px;
            font-size: 1.3em;
        }

        /* Color scheme */
        .trigger {
            background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
        }

        .job {
            background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);
        }

        .step {
            background: linear-gradient(135deg, #43e97b 0%, #38f9d7 100%);
        }

        .runner {
            background: linear-gradient(135deg, #fa709a 0%, #fee140 100%);
        }

        .action {
            background: linear-gradient(135deg, #30cfd0 0%, #330867 100%);
        }

        .secret {
            background: linear-gradient(135deg, #ff6b6b 0%, #ee5a6f 100%);
        }

        /* Flow diagram */
        .flow-diagram {
            display: flex;
            flex-direction: column;
            gap: 15px;
            margin-top: 20px;
        }

        .flow-step {
            display: flex;
            align-items: center;
            gap: 15px;
            padding: 15px;
            background: #f8f9fa;
            border-radius: 8px;
            border-left: 5px solid #667eea;
        }

        .flow-number {
            background: #667eea;
            color: white;
            width: 40px;
            height: 40px;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            font-weight: bold;
            flex-shrink: 0;
        }

        .flow-content {
            flex: 1;
        }

        .flow-content strong {
            color: #667eea;
        }

        /* Code blocks */
        .code-block {
            background: #1e1e1e;
            color: #d4d4d4;
            padding: 15px;
            border-radius: 8px;
            overflow-x: auto;
            font-family: 'Courier New', monospace;
            font-size: 0.9em;
            margin: 15px 0;
            border-left: 4px solid #00f2fe;
        }

        .code-block.example {
            border-left-color: #43e97b;
        }

        .code-block.warning {
            border-left-color: #f5576c;
        }

        /* Comparison boxes */
        .comparison {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 20px;
            margin-top: 20px;
        }

        .comparison-item {
            padding: 20px;
            border-radius: 10px;
        }

        .wrong {
            background: #ffe5e5;
            border: 2px solid #f5576c;
        }

        .correct {
            background: #e5f5e5;
            border: 2px solid #43e97b;
        }

        .comparison-item h4 {
            margin-bottom: 10px;
            font-size: 1.1em;
        }

        .wrong h4 {
            color: #f5576c;
        }

        .correct h4 {
            color: #43e97b;
        }

        /* Interactive demo */
        .interactive {
            background: #f8f9fa;
            padding: 20px;
            border-radius: 10px;
            margin-top: 20px;
        }

        button {
            background: #667eea;
            color: white;
            border: none;
            padding: 12px 24px;
            border-radius: 6px;
            cursor: pointer;
            font-size: 1em;
            transition: background 0.3s ease;
            margin: 5px;
        }

        button:hover {
            background: #764ba2;
        }

        .demo-output {
            background: #1e1e1e;
            color: #00f2fe;
            padding: 15px;
            border-radius: 8px;
            margin-top: 15px;
            font-family: monospace;
            min-height: 100px;
            max-height: 300px;
            overflow-y: auto;
        }

        /* Emoji and icons */
        .emoji {
            font-size: 1.5em;
            margin-right: 8px;
        }

        /* Checklist */
        .checklist {
            list-style: none;
            margin-top: 15px;
        }

        .checklist li {
            padding: 10px;
            margin: 8px 0;
            background: #f8f9fa;
            border-radius: 6px;
            border-left: 4px solid #667eea;
        }

        .checklist input[type="checkbox"] {
            margin-right: 10px;
            cursor: pointer;
            width: 18px;
            height: 18px;
        }

        /* Responsive */
        @media (max-width: 768px) {
            h1 {
                font-size: 1.8em;
            }

            .comparison {
                grid-template-columns: 1fr;
            }

            .concept-box {
                grid-template-columns: 1fr;
            }

            .section {
                padding: 20px;
            }
        }

        /* Footer */
        footer {
            text-align: center;
            color: white;
            padding: 20px;
            margin-top: 40px;
        }

        .badge {
            display: inline-block;
            background: rgba(255,255,255,0.2);
            padding: 8px 16px;
            border-radius: 20px;
            margin: 5px;
            color: white;
        }

        /* Highlight important */
        .highlight {
            background: #fff3cd;
            padding: 15px;
            border-left: 4px solid #ffc107;
            border-radius: 4px;
            margin: 15px 0;
        }

        .highlight strong {
            color: #ff6b6b;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🚀 GitHub Actions Visual Guide</h1>
        <p class="subtitle">Understanding CI/CD for Beginners (Grade 8-9 Level)</p>

        <!-- Section 1: What is GitHub Actions -->
        <div class="section">
            <h2><span class="emoji">🤔</span>What is GitHub Actions?</h2>
            <p>Imagine you have homework:</p>
            
            <div class="comparison">
                <div class="comparison-item wrong">
                    <h4><span class="emoji">❌</span>Without Automation</h4>
                    <ol>
                        <li>Write code</li>
                        <li>Test manually ⏱️</li>
                        <li>Check for errors ⏱️</li>
                        <li>Compile ⏱️</li>
                        <li>Upload ⏱️</li>
                        <li>Hope it works 😰</li>
                    </ol>
                </div>
                <div class="comparison-item correct">
                    <h4><span class="emoji">✅</span>With GitHub Actions</h4>
                    <ol>
                        <li>Write code</li>
                        <li>Push to GitHub</li>
                        <li><strong>Everything else is AUTOMATIC!</strong> 🤖</li>
                        <li>Watch it work ✨</li>
                    </ol>
                </div>
            </div>

            <div class="highlight">
                <strong>Think of it like:</strong> An automated factory assembly line that tests, builds, and deploys your code automatically!
            </div>
        </div>

        <!-- Section 2: Core Concepts -->
        <div class="section">
            <h2><span class="emoji">🧠</span>6 Core Concepts</h2>
            
            <div class="concept-box">
                <div class="concept trigger">
                    <h3>1️⃣ Trigger</h3>
                    <p><strong>What starts it?</strong></p>
                    <p>A button that starts the whole process when something happens (like pushing code).</p>
                    <div class="code-block example">
on:
  push:
    branches: [main]
                    </div>
                </div>

                <div class="concept job">
                    <h3>2️⃣ Job</h3>
                    <p><strong>What gets done?</strong></p>
                    <p>A checklist of tasks that must be completed (like making breakfast).</p>
                    <div class="code-block example">
jobs:
  build:
    steps: [...]
                    </div>
                </div>

                <div class="concept step">
                    <h3>3️⃣ Step</h3>
                    <p><strong>Individual action?</strong></p>
                    <p>One small task within a job (like "boil water" in breakfast).</p>
                    <div class="code-block example">
- run: npm install
- run: npm build
                    </div>
                </div>

                <div class="concept runner">
                    <h3>4️⃣ Runner</h3>
                    <p><strong>Who does the work?</strong></p>
                    <p>A temporary computer GitHub gives you to run your tasks.</p>
                    <div class="code-block example">
runs-on: ubuntu-latest
                    </div>
                </div>

                <div class="concept action">
                    <h3>5️⃣ Action</h3>
                    <p><strong>Pre-made recipe?</strong></p>
                    <p>Code someone else wrote for common tasks (don't reinvent the wheel!).</p>
                    <div class="code-block example">
uses: actions/checkout@v4
uses: actions/setup-node@v4
                    </div>
                </div>

                <div class="concept secret">
                    <h3>6️⃣ Secret</h3>
                    <p><strong>Hide passwords?</strong></p>
                    <p>Safe place to store sensitive info (database passwords, API keys).</p>
                    <div class="code-block example">
env:
  DB_PASS: ${{ secrets.PASSWORD }}
                    </div>
                </div>
            </div>
        </div>

        <!-- Section 3: The Flow -->
        <div class="section">
            <h2><span class="emoji">🔄</span>What Happens When You Push Code?</h2>
            
            <div class="flow-diagram">
                <div class="flow-step">
                    <div class="flow-number">1</div>
                    <div class="flow-content">
                        <strong>You push code to GitHub</strong>
                        <p>$ git push origin main</p>
                    </div>
                </div>

                <div class="flow-step">
                    <div class="flow-number">2</div>
                    <div class="flow-content">
                        <strong>GitHub detects trigger</strong>
                        <p>New code on main branch detected! 🎯</p>
                    </div>
                </div>

                <div class="flow-step">
                    <div class="flow-number">3</div>
                    <div class="flow-content">
                        <strong>GitHub starts BUILD JOB</strong>
                        <p>Fresh Ubuntu computer starts up...</p>
                    </div>
                </div>

                <div class="flow-step">
                    <div class="flow-number">4</div>
                    <div class="flow-content">
                        <strong>Steps run one by one</strong>
                        <p>✓ Download code ✓ Install Node.js ✓ Build app</p>
                    </div>
                </div>

                <div class="flow-step">
                    <div class="flow-number">5</div>
                    <div class="flow-content">
                        <strong>Build succeeds? ✅ YES</strong>
                        <p>Continue to DEPLOY JOB...</p>
                    </div>
                </div>

                <div class="flow-step">
                    <div class="flow-number">6</div>
                    <div class="flow-content">
                        <strong>GitHub connects to YOUR server via SSH</strong>
                        <p>Special secure connection using SSH key</p>
                    </div>
                </div>

                <div class="flow-step">
                    <div class="flow-number">7</div>
                    <div class="flow-content">
                        <strong>Updates your app on server</strong>
                        <p>✓ Pull code ✓ Install packages ✓ Build ✓ Restart</p>
                    </div>
                </div>

                <div class="flow-step">
                    <div class="flow-number">8</div>
                    <div class="flow-content">
                        <strong>🎉 Done! Your app is updated</strong>
                        <p>Users see the new features!</p>
                    </div>
                </div>
            </div>
        </div>

        <!-- Section 4: Secrets Explained -->
        <div class="section">
            <h2><span class="emoji">🔐</span>Secrets - Keep Passwords Safe</h2>
            
            <p><strong>What are secrets?</strong> Private information stored safely by GitHub.</p>

            <div class="comparison">
                <div class="comparison-item wrong">
                    <h4><span class="emoji">❌</span>BAD - Hard-coded in code</h4>
                    <div class="code-block warning">
const password = "MyDatabase123";
// Anyone can see this!
// GitHub is public
// Hackers will find it
                    </div>
                </div>
                <div class="comparison-item correct">
                    <h4><span class="emoji">✅</span>GOOD - Use GitHub Secrets</h4>
                    <div class="code-block example">
env:
  DB_PASSWORD: ${{ secrets.DB_PASSWORD }}

// In code:
const password = process.env.DB_PASSWORD;
// Safe! No one sees the real value
                    </div>
                </div>
            </div>

            <div class="highlight">
                <strong>How to set secrets:</strong>
                <ol>
                    <li>Go to GitHub repo → Settings</li>
                    <li>Click "Secrets and variables" → "Actions"</li>
                    <li>Click "New repository secret"</li>
                    <li>Enter name and value</li>
                    <li>Click "Add secret" ✅</li>
                </ol>
            </div>
        </div>

        <!-- Section 5: Interactive Demo -->
        <div class="section">
            <h2><span class="emoji">🎮</span>Try It: Simulate a Deployment</h2>
            
            <p>Click the button to see what happens during deployment:</p>
            
            <div class="interactive">
                <button onclick="simulateWorkflow()">▶️ Start Deployment</button>
                <button onclick="clearOutput()">🔄 Clear Log</button>
                
                <div class="demo-output" id="demoOutput">
Ready to start... Click "Start Deployment" button ⬆️
                </div>
            </div>
        </div>

        <!-- Section 6: Common Mistakes -->
        <div class="section">
            <h2><span class="emoji">⚠️</span>Common Mistakes</h2>
            
            <div class="comparison">
                <div class="comparison-item wrong">
                    <h4>❌ Mistake 1: No Secrets</h4>
                    <div class="code-block warning">
env:
  DATABASE_URL: mongodb://user:pass@localhost
  JWT_KEY: super_secret_123
// Passwords exposed!
                    </div>
                </div>
                <div class="comparison-item correct">
                    <h4>✅ Fix: Use Secrets</h4>
                    <div class="code-block example">
env:
  DATABASE_URL: ${{ secrets.DATABASE_URL }}
  JWT_KEY: ${{ secrets.JWT_KEY }}
// Safe!
                    </div>
                </div>
            </div>

            <div class="comparison">
                <div class="comparison-item wrong">
                    <h4>❌ Mistake 2: Deploy Before Build</h4>
                    <div class="code-block warning">
jobs:
  build: ...
  deploy:
    # No 'needs: build'
    # Deploy might start before build finishes!
                    </div>
                </div>
                <div class="comparison-item correct">
                    <h4>✅ Fix: Use Dependency</h4>
                    <div class="code-block example">
jobs:
  build: ...
  deploy:
    needs: build  # Wait for build!
    # Now we're safe
                    </div>
                </div>
            </div>
        </div>

        <!-- Section 7: Checklist -->
        <div class="section">
            <h2><span class="emoji">✅</span>Before First Deployment</h2>
            
            <ul class="checklist">
                <li><input type="checkbox"> GitHub Secrets configured (EC2_HOST, EC2_USER, EC2_SSH_KEY)</li>
                <li><input type="checkbox"> EC2 instance is running</li>
                <li><input type="checkbox"> SSH port (22) allowed in security group</li>
                <li><input type="checkbox"> Node.js installed on EC2 (v20)</li>
                <li><input type="checkbox"> npm symlinks created (/usr/local/bin/npm)</li>
                <li><input type="checkbox"> Workflow file at .github/workflows/ci.yml</li>
                <li><input type="checkbox"> YAML syntax is correct (indentation!)</li>
                <li><input type="checkbox"> package.json has "build" script</li>
                <li><input type="checkbox"> dist/index.js exists after build</li>
                <li><input type="checkbox"> PM2 installed globally</li>
            </ul>
        </div>

        <!-- Section 8: FAQ -->
        <div class="section">
            <h2><span class="emoji">🤔</span>Frequently Asked Questions</h2>
            
            <div style="display: grid; gap: 20px; margin-top: 20px;">
                <div style="border-left: 4px solid #667eea; padding-left: 15px;">
                    <h4>Q: What if the workflow breaks my app?</h4>
                    <p>A: You can stop PM2 and rollback instantly. Deploy and rollback takes seconds.</p>
                </div>

                <div style="border-left: 4px solid #667eea; padding-left: 15px;">
                    <h4>Q: How often should I deploy?</h4>
                    <p>A: Every time you push! The automation ensures quality.</p>
                </div>

                <div style="border-left: 4px solid #667eea; padding-left: 15px;">
                    <h4>Q: Can I see the logs?</h4>
                    <p>A: Yes! Go to Actions → Click workflow → See all output.</p>
                </div>

                <div style="border-left: 4px solid #667eea; padding-left: 15px;">
                    <h4>Q: Is this secure?</h4>
                    <p>A: Yes! If you use Secrets correctly (which you should). Passwords never in code.</p>
                </div>

                <div style="border-left: 4px solid #667eea; padding-left: 15px;">
                    <h4>Q: What if something fails?</h4>
                    <p>A: Check the error message in Actions tab. SSH to server and debug manually.</p>
                </div>
            </div>
        </div>

        <!-- Section 9: Summary -->
        <div class="section" style="background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); color: white;">
            <h2 style="color: white; border-bottom-color: white;">🎯 Summary</h2>
            
            <div style="display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 20px; margin-top: 20px;">
                <div style="background: rgba(255,255,255,0.1); padding: 15px; border-radius: 8px;">
                    <h3>🚀 Automation</h3>
                    <p>GitHub Actions = Robots that do your work</p>
                </div>

                <div style="background: rgba(255,255,255,0.1); padding: 15px; border-radius: 8px;">
                    <h3>🔄 Continuous</h3>
                    <p>Every push = Automatic test & deploy</p>
                </div>

                <div style="background: rgba(255,255,255,0.1); padding: 15px; border-radius: 8px;">
                    <h3>🔐 Secure</h3>
                    <p>Secrets keep passwords safe</p>
                </div>

                <div style="background: rgba(255,255,255,0.1); padding: 15px; border-radius: 8px;">
                    <h3>⚡ Fast</h3>
                    <p>Deploy in minutes, not hours</p>
                </div>

                <div style="background: rgba(255,255,255,0.1); padding: 15px; border-radius: 8px;">
                    <h3>✅ Reliable</h3>
                    <p>Same steps every time = fewer mistakes</p>
                </div>

                <div style="background: rgba(255,255,255,0.1); padding: 15px; border-radius: 8px;">
                    <h3>💪 Scalable</h3>
                    <p>Works for small and large projects</p>
                </div>
            </div>
        </div>

        <!-- Final Message -->
        <div class="section">
            <h2 style="text-align: center;">🌟 You've Got This!</h2>
            <p style="text-align: center; font-size: 1.1em;">
                GitHub Actions seems complex at first, but remember:
            </p>
            <div class="highlight" style="text-align: center; margin-top: 20px;">
                <p><strong>When → What → How → Where → Why</strong></p>
                <p style="margin-top: 10px; font-size: 0.95em;">
                    <strong>When:</strong> Code pushed to main<br>
                    <strong>What:</strong> Run these steps<br>
                    <strong>How:</strong> Using environment variables<br>
                    <strong>Where:</strong> On GitHub runner, then SSH to server<br>
                    <strong>Why:</strong> Automate boring work, deploy safely
                </p>
            </div>

            <p style="text-align: center; margin-top: 20px; font-size: 1.1em;">
                Start small, understand each step, then build bigger workflows. 
                <br>
                <strong>Every expert was once a beginner who didn't give up.</strong>
            </p>
        </div>

        <footer>
            <p>🚀 Ready to deploy? You've got all the knowledge you need!</p>
            <div style="margin-top: 15px;">
                <span class="badge">✨ Beginner-Friendly</span>
                <span class="badge">🎨 Visual Learning</span>
                <span class="badge">📚 Comprehensive</span>
                <span class="badge">🎯 Practical</span>
            </div>
        </footer>
    </div>

    <script>
        function simulateWorkflow() {
            const output = document.getElementById('demoOutput');
            output.innerHTML = '';
            
            const steps = [
                '⏳ Waiting for trigger...',
                '✅ Code pushed to main branch detected!',
                '🚀 Starting BUILD job...',
                '📥 Downloading code from GitHub',
                '📦 Installing Node.js v20',
                '📚 Installing npm packages',
                '🔨 Building TypeScript → JavaScript',
                '✅ Build successful!',
                '🚀 Starting DEPLOY job...',
                '🔐 SSH authentication with EC2',
                '⬇️ Downloading latest code',
                '📦 Installing packages',
                '🔨 Building application',
                '♻️ Reloading PM2 process',
                '💾 Saving PM2 state',
                '✅ SSH connection closed',
                '🎉 Deployment complete!',
                '🌐 App updated for users!'
            ];
            
            let delay = 0;
            steps.forEach((step, index) => {
                setTimeout(() => {
                    output.innerHTML += step + '\n';
                    output.scrollTop = output.scrollHeight;
                }, delay);
                delay += 400;
            });
        }
        
        function clearOutput() {
            document.getElementById('demoOutput').innerHTML = 'Log cleared... Ready for new deployment simulation!';
        }

        // Add click handlers to checkboxes
        document.querySelectorAll('.checklist input[type="checkbox"]').forEach(checkbox => {
            checkbox.addEventListener('change', function() {
                if (this.checked) {
                    this.parentElement.style.opacity = '0.6';
                    this.parentElement.style.textDecoration = 'line-through';
                } else {
                    this.parentElement.style.opacity = '1';
                    this.parentElement.style.textDecoration = 'none';
                }
            });
        });
    </script>
</body>
</html>