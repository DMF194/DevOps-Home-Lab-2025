# Kubernetes Networking: Make Your App Accessible from the Internet

*Learn to route internet traffic to your Kubernetes application using production-grade networking*

## 🎯 **What You'll Learn**

By the end of this tutorial, you'll know how to:
- **Route internet traffic** to your Kubernetes services
- **Set up custom domains** for your application
- **Configure load balancing** for high availability
- **Handle SSL/TLS** for secure connections
- **Debug networking issues** like a professional

## ⏱️ **Time Required: 20-40 minutes**

## Why This Matters

An Ingress Controller is like the front door to your application. It routes internet traffic to your services, handles SSL certificates, and provides load balancing. This is how real applications become accessible to users worldwide.

**What this means for you**: Every production application needs networking. Learning Ingress teaches you how companies like Netflix and Airbnb make their services accessible to millions of users.

ℹ️ **Simple Explanation:** An Ingress Controller is like a smart traffic director. It looks at incoming requests (like "go to gameapp.local") and routes them to the right service in your Kubernetes cluster.

## Do This

### Step 1: Set Up Ingress Controller for External Access

```bash
# Install nginx-ingress controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/baremetal/deploy.yaml
```

**Expected Output:**
```bash
namespace/ingress-nginx created
serviceaccount/ingress-nginx created
configmap/ingress-nginx-controller created
clusterrole.rbac.authorization.k8s.io/ingress-nginx created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
service/ingress-nginx-controller-admission created
service/ingress-nginx-controller created
deployment.apps/ingress-nginx-controller created
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission created
```

```bash
# Wait for ingress controller to be ready
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=120s
```

**Expected Output:**
```bash
pod/ingress-nginx-controller-xxx condition met
```

```bash
# Configure ingress controller to use hostPorts for k3d compatibility
kubectl patch deployment ingress-nginx-controller -n ingress-nginx -p '{"spec":{"template":{"spec":{"containers":[{"name":"controller","ports":[{"containerPort":80,"hostPort":80,"name":"http","protocol":"TCP"},{"containerPort":443,"hostPort":443,"name":"https","protocol":"TCP"},{"containerPort":8443,"name":"webhook","protocol":"TCP"}]}]}}}}'
```

**Expected Output:**
```bash
deployment.apps/ingress-nginx-controller patched
```

```bash
# Wait for deployment rollout to complete
kubectl rollout status deployment/ingress-nginx-controller -n ingress-nginx
```

**Expected Output:**
```bash
deployment "ingress-nginx-controller" successfully rolled out
```

```bash
# Deploy your application's ingress rules
kubectl apply -f k8s/ingress.yaml
```

**Expected Output:**
```bash
ingress.networking.k8s.io/humor-game-ingress configured
```

```bash
# Verify ingress is configured
kubectl get ingress -n humor-game
```

**Expected Output:**
```bash
NAME                 CLASS   HOSTS           ADDRESS      PORTS   AGE
humor-game-ingress   nginx   gameapp.local   172.18.0.3   80      2m
```

### Step 2: Configure Local Domain and Test Access

**Set up local domain for development:**
```bash
# Add local domain to your hosts file
echo "127.0.0.1 gameapp.local" | sudo tee -a /etc/hosts
```

**Expected Output:**
```bash
127.0.0.1 gameapp.local
```

```bash
# Verify DNS resolution works
ping gameapp.local
# Should ping 127.0.0.1 successfully
```

**Expected Output:**
```bash
PING gameapp.local (127.0.0.1): 56 data bytes
64 bytes from gameapp.local (127.0.0.1): icmp_seq=1 time=0.037 ms
64 bytes from gameapp.local (127.0.0.1): icmp_seq=2 time=0.034 ms
64 bytes from gameapp.local (127.0.0.1): icmp_seq=3 time=0.033 ms
--- gameapp.local ping statistics ---
3 packets transmitted, 3 received, 0% packet loss
```

**Test your Kubernetes application:**
```bash
# Test frontend through Ingress
curl -H "Host: gameapp.local" -I http://localhost:8080/
```

**Expected Output:**
```bash
HTTP/1.1 200 OK
Date: Sat, 30 Aug 2025 00:33:20 GMT
Content-Type: text/html
Content-Length: 16910
Connection: keep-alive
Last-Modified: Sat, 30 Aug 2025 00:01:16 GMT
ETag: "68b23f4c-420e"
Expires: Sat, 30 Aug 2025 00:38:20 GMT
Cache-Control: max-age=300
Cache-Control: public, must-revalidate
Accept-Ranges: bytes
```
When doing in Wins11, terminal Gitbash
Note unable to get above expected output

```bash$ kubectl get pods -n ingress-nginx
NAME                                       READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-pqvlh       0/1     Completed   0          51m
ingress-nginx-admission-patch-8bqbh        0/1     Completed   0          51m
ingress-nginx-controller-b6f65647c-qb2t6   1/1     Running     0          11m

derkming.fong@RWMYOA-LTP-020 MINGW64 ~/DevOps-Home-Lab-2025 (dev)
$ kubectl get svc -n ingress-nginx
NAME                                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             NodePort    10.43.181.117   <none>        80:31257/TCP,443:30479/TCP   51m
ingress-nginx-controller-admission   ClusterIP   10.43.50.113    <none>        443/TCP                      51m

derkming.fong@RWMYOA-LTP-020 MINGW64 ~/DevOps-Home-Lab-2025 (dev)
$ kubectl get ingress -n humor-game
NAME                 CLASS   HOSTS                                           ADDRESS      PORTS   AGE
humor-game-ingress   nginx   gameapp.local,gameapp.games,app.gameapp.games   172.19.0.5   80      39m

derkming.fong@RWMYOA-LTP-020 MINGW64 ~/DevOps-Home-Lab-2025 (dev)
$ kubectl describe ingress humor-game-ingress -n humor-game
Name:             humor-game-ingress
Labels:           <none>
Namespace:        humor-game
Address:          172.19.0.5
Ingress Class:    nginx
Default backend:  <default>
Rules:
  Host               Path  Backends
  ----               ----  --------
  gameapp.local
                     /api       backend:3001 (10.42.1.7:3001)
                     /health    backend:3001 (10.42.1.7:3001)
                     /metrics   backend:3001 (10.42.1.7:3001)
                     /debug     backend:3001 (10.42.1.7:3001)
                     /          frontend:80 (10.42.2.5:80)
  gameapp.games
                     /api       backend:3001 (10.42.1.7:3001)
                     /health    backend:3001 (10.42.1.7:3001)
                     /metrics   backend:3001 (10.42.1.7:3001)
                     /debug     backend:3001 (10.42.1.7:3001)
                     /          frontend:80 (10.42.2.5:80)
  app.gameapp.games
                     /api       backend:3001 (10.42.1.7:3001)
                     /health    backend:3001 (10.42.1.7:3001)
                     /metrics   backend:3001 (10.42.1.7:3001)
                     /debug     backend:3001 (10.42.1.7:3001)
                     /          frontend:80 (10.42.2.5:80)
Annotations:         nginx.ingress.kubernetes.io/cors-allow-credentials: true
                     nginx.ingress.kubernetes.io/cors-allow-origin: *
Events:
  Type    Reason  Age                From                      Message
  ----    ------  ----               ----                      -------
  Normal  Sync    38m (x2 over 39m)  nginx-ingress-controller  Scheduled for sync
  Normal  Sync    11m (x2 over 12m)  nginx-ingress-controller  Scheduled for sync

  $ curl -H "Host: gameapp.local" http://localhost:31257/
curl: (7) Failed to connect to localhost port 31257 after 2233 ms: Couldn't connect to server

derkming.fong@RWMYOA-LTP-020 MINGW64 ~/DevOps-Home-Lab-2025 (dev)
$ curl http://gameapp.local:31257/
curl: (7) Failed to connect to gameapp.local port 31257 after 2031 ms: Couldn't connect to server

derkming.fong@RWMYOA-LTP-020 MINGW64 ~/DevOps-Home-Lab-2025 (dev)
$ kubectl patch svc ingress-nginx-controller -n ingress-nginx -p '{"spec":{"type":"LoadBalancer"}}'
service/ingress-nginx-controller patched

derkming.fong@RWMYOA-LTP-020 MINGW64 ~/DevOps-Home-Lab-2025 (dev)
$ kubectl get svc -n ingress-nginx ingress-nginx-controller
NAME                       TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller   LoadBalancer   10.43.181.117   <pending>     80:31257/TCP,443:30479/TCP   63m

derkming.fong@RWMYOA-LTP-020 MINGW64 ~/DevOps-Home-Lab-2025 (dev)
$ curl http://gameapp.local/
curl: (7) Failed to connect to gameapp.local port 80 after 2050 ms: Couldn't connect to server

derkming.fong@RWMYOA-LTP-020 MINGW64 ~/DevOps-Home-Lab-2025 (dev)
$ kubectl get svc -n ingress-nginx ingress-nginx-controller
NAME                       TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller   LoadBalancer   10.43.181.117   <pending>     80:31257/TCP,443:30479/TCP   64m

derkming.fong@RWMYOA-LTP-020 MINGW64 ~/DevOps-Home-Lab-2025 (dev)
$ curl http://gameapp.local/
curl: (7) Failed to connect to gameapp.local port 80 after 2049 ms: Couldn't connect to server

derkming.fong@RWMYOA-LTP-020 MINGW64 ~/DevOps-Home-Lab-2025 (dev)
$ kubectl port-forward -n ingress-nginx svc/ingress-nginx-controller 8080:80
Forwarding from 127.0.0.1:8080 -> 80
Handling connection for 8080
Handling connection for 8080
Handling connection for 8080
Handling connection for 8080

```
Open in new Gitbash terminal
The NodePort can't be accessed from localhost because k3d runs in Docker containers. The LoadBalancer is stuck in <pending> because k3d needs special configuration. Let me help you fix this properly.
Solution: Access k3d via Docker Network
The ingress address shown is 172.19.0.5 - this is the Docker network IP. You need to access it through k3d's exposed ports.
```bashderkming.fong@RWMYOA-LTP-020 MINGW64 ~/DevOps-Home-Lab-2025 (dev)
$ curl http://gameapp.local:8080/
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>🎮 Humor Memory Game - DevOps Learning Edition 😂</title>
    <meta
      name="description"
      content="A fun memory card game with funny emojis and jokes, Made for DevOps learning!"
    />
    <meta name="author" content="DevOps Learning Team - Osomudeya Zudonu" />

    <!-- Favicon -->
    <link
      rel="icon"
      href="data:image/svg+xml,<svg xmlns=%22http://www.w3.org/2000/svg%22 viewBox=%220 0 100 100%22><text y=%22.9em%22 font-size=%2290%22>🎮</text></svg>"
    />

    <!-- CSS -->
    <link rel="stylesheet" href="/styles/main.css?v=20250819" />

    <!-- Configuration Script - MUST RUN BEFORE ANY OTHER SCRIPTS -->
    <script>
      console.log('🚀 HTML Configuration Script Starting...');

      // Environment-driven API configuration
      // Works in both Docker Compose (nginx proxy) and Kubernetes (ingress)
      window.API_BASE_URL = '/api';
      window.ENVIRONMENT = 'development';
      window.BUILD_TIMESTAMP = '';

      // Validation and fallbacks
      if (!window.API_BASE_URL || window.API_BASE_URL === '') {
        console.warn('⚠️ API_BASE_URL not set, using fallback /api');
        window.API_BASE_URL = '/api';
      }

      // Ensure configuration is complete before proceeding
      window.CONFIG_READY = true;

      console.log('🔧 Environment:', window.ENVIRONMENT);
      console.log('🔧 API Base URL set to:', window.API_BASE_URL);
      console.log('🔧 Build Timestamp:', window.BUILD_TIMESTAMP);
      console.log('🌐 Current hostname:', window.location.hostname);
      console.log('🔗 Current URL:', window.location.href);
      console.log('✅ Configuration loaded successfully');

      // Prevent any other scripts from overriding this
      Object.defineProperty(window, 'API_BASE_URL', {
        value: window.API_BASE_URL,
        writable: false,
        configurable: false
      });
    </script>

    <!-- Preload critical resources -->
    <link rel="preload" href="/scripts/game.js" as="script" />
  </head>
  <body>
    <!-- Loading Screen -->
    <div id="loadingScreen" class="loading-screen">
      <div class="loading-content">
        <div class="loading-spinner">🎮</div>
        <h2>Loading Humor Memory Game...</h2>
        <p>Preparing the funniest cards for you! 😂</p>
        <div class="loading-bar">
          <div class="loading-progress"></div>
        </div>
      </div>
    </div>

    <!-- Main Game Container -->
    <div id="gameContainer" class="game-container" style="display: none">
      <!-- Header -->
      <header class="game-header">
        <div class="header-content">
          <h1 class="game-title">
            <span class="title-emoji">🎮</span>
            Humor Memory Game
            <span class="title-emoji">😂</span>
          </h1>
          <p class="game-subtitle">A DevOps Learning Adventure!</p>
        </div>

        <!-- Navigation Tabs -->
        <nav class="game-nav">
          <button class="nav-btn active" data-tab="game">🎯 Game</button>
          <button class="nav-btn" data-tab="leaderboard">🏆 Leaderboard</button>
          <button class="nav-btn" data-tab="stats">📊 My Stats</button>
          <button class="nav-btn" data-tab="about">ℹ️ About</button>
        </nav>
      </header>

      <!-- Game Tab -->
      <section id="gameTab" class="tab-content active">
        <!-- User Setup -->
        <div id="userSetup" class="user-setup">
          <div class="setup-card">
            <h2>🎪 Welcome to the Memory Circus!</h2>
            <p>Enter your username to start the hilarious memory challenge!</p>

            <div class="input-group">
              <label for="usernameInput">👤 Username:</label>
              <input
                type="text"
                id="usernameInput"
                placeholder="Enter your username..."
                maxlength="50"
                autocomplete="username"
              />
            </div>

            <div class="input-group">
              <label for="difficultySelect">🎯 Difficulty:</label>
              <select id="difficultySelect">
                <option value="easy">😊 Easy (4x4 - 16 cards)</option>
                <option value="medium">🤔 Medium (5x4 - 20 cards)</option>
                <option value="hard">😤 Hard (6x4 - 24 cards)</option>
                <option value="expert">🤯 Expert (6x5 - 30 cards)</option>
              </select>
            </div>

            <div class="input-group">
              <label>🎨 Emoji Categories (optional):</label>
              <div class="category-selector">
                <label class="category-option">
                  <input type="checkbox" value="classic" /> 😂 Classic
                </label>
                <label class="category-option">
                  <input type="checkbox" value="food" /> 🍕 Food
                </label>
                <label class="category-option">
                  <input type="checkbox" value="space" /> 🚀 Space
                </label>
                <label class="category-option">
                  <input type="checkbox" value="fantasy" /> 🦄 Fantasy
                </label>
                <label class="category-option">
                  <input type="checkbox" value="tech" /> 🤖 Tech
                </label>
              </div>
            </div>

            <button id="startGameBtn" class="btn btn-primary">
              🚀 Start Game!
            </button>
          </div>
        </div>

        <!-- Game Board -->
        <div id="gameBoard" class="game-board" style="display: none">
          <!-- Game Info Bar -->
          <div class="game-info">
            <div class="info-item">
              <span class="info-label">👤 Player:</span>
              <span id="playerName" class="info-value">-</span>
            </div>
            <div class="info-item">
              <span class="info-label">🎯 Score:</span>
              <span id="gameScore" class="info-value">0</span>
            </div>
            <div class="info-item">
              <span class="info-label">🔢 Moves:</span>
              <span id="gameMoves" class="info-value">0</span>
            </div>
            <div class="info-item">
              <span class="info-label">⏱️ Time:</span>
              <span id="gameTime" class="info-value">00:00</span>
            </div>
            <div class="info-item">
              <span class="info-label">🃏 Pairs:</span>
              <span id="foundPairs" class="info-value">0</span>
              <span class="info-separator">/</span>
              <span id="totalPairs" class="info-value">8</span>
            </div>
          </div>

          <!-- Cards Grid -->
          <div id="cardsGrid" class="cards-grid">
            <!-- Cards will be dynamically generated here -->
          </div>

          <!-- Game Controls -->
          <div class="game-controls">
            <button id="pauseBtn" class="btn btn-secondary">⏸️ Pause</button>
            <button id="newGameBtn" class="btn btn-primary">🔄 New Game</button>
            <button id="quitBtn" class="btn btn-danger">❌ Quit</button>
          </div>

          <!-- Message Display -->
          <div id="gameMessage" class="game-message" style="display: none">
            <div class="message-content">
              <div class="message-emoji">🎉</div>
              <div class="message-text">Great match!</div>
            </div>
          </div>
        </div>

        <!-- Game Complete Modal -->
        <div id="gameCompleteModal" class="modal" style="display: none">
          <div class="modal-content">
            <div class="modal-header">
              <h2>🎉 Game Complete! 🏆</h2>
              <button class="modal-close">&times;</button>
            </div>
            <div class="modal-body">
              <div class="completion-stats">
                <div class="stat-item">
                  <div class="stat-emoji">🎯</div>
                  <div class="stat-label">Final Score</div>
                  <div id="finalScore" class="stat-value">0</div>
                </div>
                <div class="stat-item">
                  <div class="stat-emoji">⏱️</div>
                  <div class="stat-label">Time</div>
                  <div id="finalTime" class="stat-value">00:00</div>
                </div>
                <div class="stat-item">
                  <div class="stat-emoji">🔢</div>
                  <div class="stat-label">Moves</div>
                  <div id="finalMoves" class="stat-value">0</div>
                </div>
                <div class="stat-item">
                  <div class="stat-emoji">🎪</div>
                  <div class="stat-label">Accuracy</div>
                  <div id="finalAccuracy" class="stat-value">0%</div>
                </div>
              </div>

              <div id="performanceRating" class="performance-rating">
                <div class="rating-emoji">⭐</div>
                <div class="rating-text">Great Job!</div>
              </div>

              <div
                id="achievementsList"
                class="achievements-list"
                style="display: none"
              >
                <h3>🏆 Achievements Unlocked!</h3>
                <div class="achievements-container">
                  <!-- Achievements will be populated here -->
                </div>
              </div>
            </div>
            <div class="modal-footer">
              <button id="playAgainBtn" class="btn btn-primary">
                🎮 Play Again
              </button>
              <button id="viewLeaderboardBtn" class="btn btn-secondary">
                🏆 Leaderboard
              </button>
            </div>
          </div>
        </div>
      </section>

      <!-- Leaderboard Tab -->
      <section id="leaderboardTab" class="tab-content">
        <div class="leaderboard-container">
          <h2>🏆 Memory Champions Leaderboard</h2>

          <div class="leaderboard-controls">
            <select id="leaderboardFilter">
              <option value="all">🌟 All Time</option>
              <option value="week">📅 This Week</option>
              <option value="month">📆 This Month</option>
            </select>
            <button id="refreshLeaderboard" class="btn btn-secondary">
              🔄 Refresh
            </button>
          </div>

          <div id="leaderboardContent" class="leaderboard-content">
            <div class="loading-spinner">Loading leaderboard... 📊</div>
          </div>
        </div>
      </section>

      <!-- Stats Tab -->
      <section id="statsTab" class="tab-content">
        <div class="stats-container">
          <h2>📊 Your Performance Statistics</h2>

          <div id="userStatsContent" class="user-stats-content">
            <div class="stats-placeholder">
              <div class="placeholder-emoji">📈</div>
              <p>Play a game to see your stats here!</p>
              <button class="btn btn-primary" onclick="switchTab('game')">
                🎮 Start Playing
              </button>
            </div>
          </div>
        </div>
      </section>

      <!-- About Tab -->
      <section id="aboutTab" class="tab-content">
        <div class="about-container">
          <h2>ℹ️ About Humor Memory Game</h2>

          <div class="about-content">
            <div class="about-section">
              <h3>🎮 What is this game?</h3>
              <p>
                A fun memory card matching game featuring hilarious emojis and
                jokes! Perfect for testing your memory skills while having a
                good laugh.
              </p>
            </div>

            <div class="about-section">
              <h3>🎯 How to Play</h3>
              <ol>
                <li>🎪 Enter your username and select difficulty</li>
                <li>🃏 Click cards to flip them and find matching pairs</li>
                <li>⚡ Match cards quickly for bonus points!</li>
                <li>🏆 Complete all pairs to finish the game</li>
                <li>📊 Check the leaderboard to see your ranking!</li>
              </ol>
            </div>

            <div class="about-section">
              <h3>🔥 Scoring System</h3>
              <ul>
                <li>🎯 Base points per match (varies by difficulty)</li>
                <li>⚡ Speed bonus for quick matches</li>
                <li>💎 Perfect game bonus (no wrong moves)</li>
                <li>🔥 Streak bonus for consecutive matches</li>
              </ul>
            </div>

            <div class="about-section">
              <h3>🛠️ Built for DevOps Learning</h3>
              <p>This application demonstrates:</p>
              <ul>
                <li>🐳 Docker containerization</li>
                <li>🗄️ PostgreSQL database integration</li>
                <li>⚡ Redis caching</li>
                <li>🌐 RESTful API design</li>
                <li>☁️ Cloud-ready architecture</li>
                <li>🔒 Security best practices</li>
              </ul>
            </div>

            <div class="about-section">
              <h3>💻 Tech Stack</h3>
              <div class="tech-stack">
                <span class="tech-badge">Node.js</span>
                <span class="tech-badge">Express</span>
                <span class="tech-badge">PostgreSQL</span>
                <span class="tech-badge">Redis</span>
                <span class="tech-badge">Docker</span>
                <span class="tech-badge">HTML5</span>
                <span class="tech-badge">CSS3</span>
                <span class="tech-badge">JavaScript</span>
              </div>
            </div>

            <div class="about-section">
              <h3>🎉 Fun Facts</h3>
              <ul>
                <li>😂 Over 30 different funny emojis to match!</li>
                <li>
                  🎭 Random success and failure messages for entertainment
                </li>
                <li>🏆 Real-time leaderboard with global rankings</li>
                <li>📊 Detailed statistics and achievement system</li>
                <li>⚡ Optimized with Redis caching for speed</li>
                <li>🔒 Secure and scalable architecture</li>
              </ul>
            </div>
          </div>
        </div>
      </section>
    </div>

    <!-- Error Modal -->
    <div id="errorModal" class="modal error-modal" style="display: none">
      <div class="modal-content">
        <div class="modal-header">
          <h2>😅 Oops! Something went wrong</h2>
          <button class="modal-close">&times;</button>
        </div>
        <div class="modal-body">
          <div class="error-content">
            <div class="error-emoji">🤔</div>
            <div id="errorMessage" class="error-message">
              An unexpected error occurred. Please try again!
            </div>
            <div class="error-suggestion">
              💡 Try refreshing the page or check your internet connection.
            </div>
          </div>
        </div>
        <div class="modal-footer">
          <button id="retryBtn" class="btn btn-primary">🔄 Try Again</button>
          <button class="modal-close btn btn-secondary">❌ Close</button>
        </div>
      </div>
    </div>

    <!-- Notification Toast -->
    <div id="notification" class="notification" style="display: none">
      <div class="notification-content">
        <div class="notification-emoji">ℹ️</div>
        <div class="notification-message">Notification message</div>
      </div>
    </div>

    <!-- Footer -->
    <footer class="game-footer">
      <div class="footer-content">
        <p>
          🎮 Humor Memory Game v1.0 | Built with ❤️ for DevOps Learning |
          <a
            href="https://github.com/your-org/humor-memory-game"
            target="_blank"
            >📖 View on GitHub</a
          >
        </p>
        <p class="footer-subtitle">
          🚀 Perfect for learning Docker, Kubernetes, CI/CD, and cloud
          deployment!
        </p>
      </div>
    </footer>

    <!-- JavaScript -->
    <script src="/scripts/game.js?v=20250821"></script>

    <!-- Service Worker for PWA (optional) -->
    <script>
      // Register service worker for offline capability
      if ('serviceWorker' in navigator) {
        navigator.serviceWorker
          .register('/sw.js')
          .then((registration) =>
            console.log('🔧 SW registered:', registration)
          )
          .catch((error) => console.log('❌ SW registration failed:', error));
      }
    </script>
  </body>
</html>

derkming.fong@RWMYOA-LTP-020 MINGW64 ~/DevOps-Home-Lab-2025 (dev)
$ curl http://gameapp.local:8080/health
{"status":"healthy","timestamp":"2026-01-12T09:33:21.255Z","services":{"database":"connected","redis":"connected","api":"running"},"version":"1.0.0","environment":"development"}
derkming.fong@RWMYOA-LTP-020 MINGW64 ~/DevOps-Home-Lab-2025 (dev)
$ curl http://gameapp.local:8080/api
{"message":"Welcome to the Humor Memory Game API! 🎮😂","version":"1.0.0","endpoints":{"game":{"POST /api/game/start":"Start a new game session","POST /api/game/match":"Submit a card match","POST /api/game/complete":"Complete a game and save score","GET /api/game/:gameId":"Get game details"},"scores":{"GET /api/scores/:username":"Get user scores and stats","POST /api/scores/user":"Create or update user"},"leaderboard":{"GET /api/leaderboard":"Get top players (cached)","GET /api/leaderboard/fresh":"Get fresh leaderboard data"}},"health":"/api/health","documentation":"API-only backend for separated architecture! 🎯"}
derkming.fong@RWMYOA-LTP-020 MINGW64 ~/DevOps-Home-Lab-2025 (dev)
$ curl http://gameapp.local:8080/metrics
# HELP process_cpu_user_seconds_total Total user CPU time spent in seconds.
# TYPE process_cpu_user_seconds_total counter
process_cpu_user_seconds_total 28.531684


```

```bash
# Test API health through Ingress
curl -H "Host: gameapp.local" http://localhost:8080/api/health
```

**Expected Output:**
```json
{
  "status": "healthy",
  "timestamp": "2025-08-30T00:33:26.867Z",
  "services": {
    "database": "connected",
    "redis": "connected",
    "api": "running"
  },
  "version": "1.0.0",
  "environment": "development"
}
```

```bash
# Open in browser with domain
open http://gameapp.local:8080
```

### Step 3: Verify Ingress Configuration

```bash
# Check ingress status
kubectl get ingress -n humor-game
# Should show: humor-game-ingress with nginx class
```

**Expected Output:**
```bash
NAME                 CLASS   HOSTS           ADDRESS      PORTS   AGE
humor-game-ingress   nginx   gameapp.local   172.18.0.3   80      13m
```

```bash
# Check ingress controller pods
kubectl get pods -n ingress-nginx
# Should show: nginx-ingress-controller pod with "1/1 Running"
```

**Expected Output:**
```bash
NAME                                        READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-md6pr        0/1     Completed   0          23m
ingress-nginx-admission-patch-b2rx9         0/1     Completed   0          23m
ingress-nginx-controller-5445788fcd-qn4x2   1/1     Running     0          73s
```

```bash
# Verify ingress rules
kubectl describe ingress humor-game-ingress -n humor-game
# Should show rules for gameapp.local
```

### Step 4: Test Full Application Functionality

```bash
# Test frontend loads
curl -H "Host: gameapp.local" -I http://localhost:8080/
# Should return: HTTP/1.1 200 OK
```

**Expected Output:**
```bash
HTTP/1.1 200 OK
Date: Sat, 30 Aug 2025 00:33:20 GMT
Content-Type: text/html
Content-Length: 16910
Connection: keep-alive
Last-Modified: Sat, 30 Aug 2025 00:01:16 GMT
ETag: "68b23f4c-420e"
Expires: Sat, 30 Aug 2025 00:38:20 GMT
Cache-Control: max-age=300
Cache-Control: public, must-revalidate
Accept-Ranges: bytes
```

```bash
# Test API endpoints
curl -H "Host: gameapp.local" http://localhost:8080/api/health
# Should return: {"status":"healthy",...}
```

```bash
# Test metrics endpoint
curl -H "Host: gameapp.local" http://localhost:8080/metrics
# Should return Prometheus metrics
```

## You Should See...

**Ingress Status:**
```bash
NAME                 CLASS   HOSTS           ADDRESS      PORTS   AGE
humor-game-ingress   nginx   gameapp.local   172.18.0.3   80      13m
```

**Ingress Controller Status:**
```bash
NAME                                        READY   STATUS      RESTARTS   AGE
ingress-nginx-controller-5445788fcd-qn4x2   1/1     Running     0          73s
```

**Host Resolution:**
```bash
PING gameapp.local (127.0.0.1): 56 data bytes
64 bytes from gameapp.local (127.0.0.1): icmp_seq=1 time=0.037 ms
```

**Frontend Response:**
```bash
HTTP/1.1 200 OK
Content-Type: text/html
```

**API Health Response:**
```json
{
  "status": "healthy",
  "services": {
    "database": "connected",
    "redis": "connected",
    "api": "running"
  },
  "timestamp": "2025-08-30T00:33:26.867Z"
}
```

## ✅ Checkpoint

Your Ingress setup is working when:
- ✅ Ingress controller pods are running in ingress-nginx namespace
- ✅ Ingress rules are configured for gameapp.local
- ✅ Frontend loads at `http://gameapp.local:8080` through Ingress
- ✅ Backend API responds to health checks through Ingress
- ✅ Ingress routes traffic correctly to both frontend and backend

## If It Fails

### Symptom: Ingress controller pods not starting
**Cause:** Resource constraints or image pull issues
**Command to confirm:** `kubectl get pods -n ingress-nginx`
**Fix:**
```bash
# Check ingress controller logs
kubectl logs -n ingress-nginx -l app.kubernetes.io/component=controller

# Check resource usage
kubectl top nodes

# If resources are low, use minimal cluster
k3d cluster delete dev-cluster
k3d cluster create dev-cluster --servers 1 --agents 1 --k3s-arg --disable=traefik@server:0
```

### Symptom: Ingress not accessible through k3d load balancer
**Cause:** Ingress controller not configured with hostPorts for k3d compatibility
**Command to confirm:** `curl -H "Host: gameapp.local" -I http://localhost:8080/`
**Fix:**
```bash
# Configure ingress controller to use hostPorts
kubectl patch deployment ingress-nginx-controller -n ingress-nginx -p '{"spec":{"template":{"spec":{"containers":[{"name":"controller","ports":[{"containerPort":80,"hostPort":80,"name":"http","protocol":"TCP"},{"containerPort":443,"hostPort":443,"name":"https","protocol":"TCP"},{"containerPort":8443,"name":"webhook","protocol":"TCP"}]}]}}}}'

# Wait for rollout
kubectl rollout status deployment/ingress-nginx-controller -n ingress-nginx

# Test again
curl -H "Host: gameapp.local" -I http://localhost:8080/
```

### Symptom: SSL certificate errors in ingress controller logs
**Cause:** Ingress configured with SSL but certificates don't exist
**Command to confirm:** `kubectl logs -n ingress-nginx -l app.kubernetes.io/component=controller | grep "SSL"`
**Fix:**
```bash
# Comment out SSL configurations in k8s/ingress.yaml
# Remove or comment: cert-manager.io/cluster-issuer, tls sections, ssl-redirect

# Apply updated ingress
kubectl apply -f k8s/ingress.yaml

# Check logs again
kubectl logs -n ingress-nginx -l app.kubernetes.io/component=controller | grep "SSL"
```

## 💡 **Reset/Rollback Commands**

If you need to start over or fix issues:

```bash
# Remove ingress rules
kubectl delete ingress humor-game-ingress -n humor-game

# Remove ingress controller (nuclear option)
kubectl delete namespace ingress-nginx

# Reset hosts file
sudo sed -i '/gameapp.local/d' /etc/hosts

# Restart ingress controller
kubectl rollout restart deployment/ingress-nginx-controller -n ingress-nginx

# Check ingress controller status
kubectl get pods -n ingress-nginx
kubectl logs -n ingress-nginx -l app.kubernetes.io/component=controller
```

## Understanding the URL Patterns

The documentation shows different URLs for different purposes:

- **`localhost:8080`** - For direct service testing and curl commands with Host headers
- **`gameapp.local:8080`** - For actual user access through the browser  
- **`Host: gameapp.local`** - For testing Ingress routing

This gives you **both development and production access patterns**:

- **Developers**: Use `localhost:8080` for direct testing and debugging
- **Users**: Access via `gameapp.local:8080` through Ingress (production-style)
- **DevOps Engineers**: Can test both patterns to verify routing works correctly

**Why both?** `localhost:8080` is the local port that k3d exposes, while `gameapp.local:8080` is the domain that Ingress routes to your services.

## What You Learned

You've implemented production networking with Ingress routing:
- **External access** to your Kubernetes application through domain names
- **Traffic routing** from Ingress controller to appropriate services
- **Production patterns** used by enterprise applications
- **Domain management** for both development and production environments
- **k3d compatibility** with hostPort configuration for ingress controllers

## Professional Skills Gained

- **Ingress controller setup** and configuration
- **Domain routing** and traffic management
- **Production networking** patterns
- **Service discovery** through external access points
- **k3d cluster integration** with ingress controllers

---

*Ingress milestone completed successfully. Application accessible via gameapp.local:8080, ready for [05-observability.md](05-observability.md).*
