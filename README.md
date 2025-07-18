# Boilerplate-RSS-bot

## RSS Bot for Webex

The RSS bot for Webex is a boilerplate parser bot designed to digest the data provided from any compatible RSS Feeds, and update a Webex space of your choosing whenever there is a new feed.

**Note:** No initial messages will be published to the spaces when app is loaded. Only when a message is received on the RSS Feed it will be published into the respective space.

## 🎯 Features

- **RSS Feed Monitoring**: Continuously monitors RSS feeds for new entries
- **Webex Integration**: Automatically posts new RSS entries to designated Webex spaces
- **Customizable Intervals**: Configurable polling intervals for RSS feeds
- **Rich Message Formatting**: Converts RSS content to HTML for better presentation
- **Robust Error Handling**: Comprehensive error handling and retry mechanisms
- **Production Ready**: Docker support with health checks and graceful shutdown
- **Comprehensive Logging**: Multiple logging destinations (Console, Syslog, Loki)
- **Proxy Support**: Built-in HTTP proxy support for enterprise environments

## 🚀 Architecture

### Core Components

- **Feed Watcher**: Monitors RSS feeds using EventEmitter pattern
- **Parser Service**: Processes RSS entries and formats messages
- **HTTP Service**: Handles Webex API communication with retry logic
- **Logger Service**: Comprehensive logging with multiple transports

### Data Flow

```
RSS Feed → Feed Watcher → Parser Service → HTTP Service → Webex Space
```

## 📋 Prerequisites

1. **Register a Bot** at [Webex Developers](https://developer.webex.com/my-apps) for your Organization, noting the Token ID
2. **Create Spaces** for Output Messages in Webex App (or skip to step 3 if you have a space/room already)
3. **Obtain RoomId** for the space/room
4. **Add Bot** created in Step 1 to your space
5. **Create a .env file** and add the required variables below

## 🔧 Installation

### Prerequisites

- **Node.js** (v16 or higher)
- **npm** or **yarn**
- **Webex Teams bot token**
- **RSS feed URL**
- **Webex room ID**

## 🚀 Deployment (Local)

1. **Clone / Download repository**
   ```bash
   git clone https://github.com/WebexSamples/Boilerplate-RSS-bot.git
   cd Boilerplate-RSS-bot
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```

3. **Create environment file**
   
   Create a `.env` file and include the required variables outlined below.
   
   - Recommend adding `CONSOLE_LEVEL=debug` during initial testing

4. **Start the integration**
   ```bash
   npm run start
   ```

5. **Review the console logs** to confirm no errors encountered

6. **Test with sample feed**
   
   As a suggestion for your tests, you can use this sample feed: https://lorem-rss.herokuapp.com/feed?unit=second&interval=30. It creates a new feed every 30 seconds.

## 🐳 Deployment (Docker)

The simplest deployment method is using [Docker](https://docs.docker.com/engine/install/) and [Docker Compose](https://docs.docker.com/compose/install/)

1. **Clone / Download repository**
   ```bash
   git clone https://github.com/WebexSamples/Boilerplate-RSS-bot.git
   cd Boilerplate-RSS-bot
   ```
   - Only docker-compose.yml is needed if using prebuilt docker image

2. **Update the included docker-compose.yml file** with the correct Environmental parameters

3. **Choose deployment method**:
   - **Use the prebuilt image** available on Docker Hub (default in docker-compose.yml)
   - **Build local image** - Uncomment build and comment image line in docker-compose.yml

4. **Provision and start the service**
   ```bash
   docker-compose up -d
   ```

5. **Review the console logs**
   ```bash
   docker logs rss-service -f
   ```
   (assuming you are using the default container name)

## ⚙️ Environment Variables

These variables can be individually defined in Docker, or loaded as an `.env` file in the `/app` directory.

### Required Variables

| Name | Type | Default | Description |
| ---- | ---- | ------- | ----------- |
| `TOKEN` | **Required** | ` ` | Bot Token for Webex Messaging Posts |
| `FEED_ROOM_ID` | **Required** | ` ` | RoomId for Webex Announcement Space |
| `RSS_FEED_URL` | **Required** | ` ` | RSS Feed URL to get the feeds |
| `RSS_INTERVAL` | Optional | `5` | Interval for RSS Checks (Seconds) |

### Logging Settings

| Name | Type | Default | Description |
| ---- | ---- | ------- | ----------- |
| `CONSOLE_LEVEL` | Optional | `info` | Logging level exposed to console |
| `APP_NAME` | Optional | `rss-service` | App Name used for logging service |
| `SYSLOG_ENABLED` | Optional | `false` | Enable external syslog server |
| `SYSLOG_HOST` | Optional | `syslog` | Destination host for syslog server |
| `SYSLOG_PORT` | Optional | `514` | Destination port for syslog server |
| `SYSLOG_PROTOCOL` | Optional | `udp4` | Destination protocol for syslog server |
| `SYSLOG_SOURCE` | Optional | `localhost` | Host to indicate that log messages are coming from |
| `LOKI_ENABLED` | Optional | `false` | Enable external Loki logging server |
| `LOKI_HOST` | Optional | `http://loki:3100` | Destination host for Loki logging server |

### HTTP Proxy

| Name | Type | Default | Description |
| ---- | ---- | ------- | ----------- |
| `GLOBAL_AGENT_HTTP_PROXY` | Optional | ` ` | Container HTTP Proxy Server (format `http://<ip or fqdn>:<port>`) |
| `GLOBAL_AGENT_NO_PROXY` | Optional | ` ` | Comma Separated List of excluded proxy domains (Supports wildcards) |
| `NODE_EXTRA_CA_CERTS` | Optional | ` ` | Include extra CA Cert bundle if required, (PEM format) ensure location is attached as a volume to the container |

## 📁 Project Structure

```
Boilerplate-RSS-bot/
├── app.js                      # Main application entry point
├── src/                        # Source code
│   ├── parserService.js       # RSS feed parsing and message formatting
│   ├── httpService.js         # HTTP client with retry logic
│   └── logger.js              # Comprehensive logging system
├── lib/                        # Library modules
│   ├── feedWatcher.js         # RSS feed monitoring class
│   └── parseRss.js            # RSS parsing utilities
├── docker-compose.yml         # Docker Compose configuration
├── Dockerfile                 # Docker image definition
├── package.json               # Node.js dependencies
├── .env                       # Environment variables (create from example)
└── README.md                  # This file
```

## 🔍 Core Modules

### Feed Watcher (`lib/feedWatcher.js`)

Event-driven RSS feed monitoring:

```javascript
const watcher = new Watcher(feedUrl, interval);

watcher.on('new entries', (entries) => {
  // Process new RSS entries
});

watcher.on('error', (error) => {
  // Handle feed errors
});
```

**Features:**
- Continuous monitoring with configurable intervals
- Automatic duplicate detection
- Chronological sorting of entries
- EventEmitter pattern for loose coupling

### Parser Service (`src/parserService.js`)

RSS content processing and formatting:

```javascript
async function parseFeed(item) {
  const output = {
    title: item.title,
    description: formatDescription(item.description),
    guid: item.guid,
    link: item.link
  };
  
  const html = `<h4><a href="${output.link}">${output.title}</a></h4>
                <p>${output.description}</p>`;
  
  await httpService.postMessage(token, roomId, html);
}
```

**Features:**
- HTML content formatting
- Description cleanup and sanitization
- Webex message posting
- Structured data extraction

### HTTP Service (`src/httpService.js`)

Robust HTTP client with advanced features:

```javascript
const axios = rateLimit(Axios.create({ timeout: 60000 }), {
  maxRPS: 5,
});

axiosRetry(axios, {
  retries: 5,
  retryDelay: exponentialDelay,
  retryCondition: customRetryLogic
});
```

**Features:**
- **Rate Limiting**: 5 requests per second maximum
- **Retry Logic**: Exponential backoff with 5 retry attempts
- **Timeout Handling**: 60-second request timeout
- **429 Handling**: Automatic retry-after header parsing
- **Connection Pooling**: Reusable HTTP connections

### Logger Service (`src/logger.js`)

Comprehensive logging system:

```javascript
const logger = require('./logger')('component');

logger.info('Information message');
logger.error('Error message');
logger.debug('Debug message');
```

**Features:**
- **Multiple Transports**: Console, Syslog, Loki
- **Structured Logging**: JSON format with timestamps
- **Log Levels**: debug, info, warn, error
- **Time Diff Tracking**: Performance monitoring
- **Component Labeling**: Source identification

## 🔐 Security Features

### Environment Variable Protection

```javascript
const env = cleanEnv(process.env, {
  TOKEN: str(),
  FEED_ROOM_ID: str(),
  RSS_FEED_URL: str(),
  RSS_INTERVAL: num({ default: 5 })
});
```

**Features:**
- **Input Validation**: Type checking and defaults
- **Required Fields**: Fails fast on missing variables
- **Sanitization**: Prevents injection attacks
- **Configuration Isolation**: Separate from application code

### HTTP Security

```javascript
const options = {
  headers: {
    authorization: `Bearer ${accessToken}`,
    'Content-Type': 'application/json',
  },
  timeout: 60000
};
```

**Features:**
- **Bearer Token Authentication**: Secure API access
- **HTTPS Only**: Encrypted communication
- **Timeout Protection**: Prevents hanging requests
- **Request Validation**: Payload verification

## 🚀 Performance Optimization

### Rate Limiting

```javascript
const axios = rateLimit(Axios.create(), {
  maxRPS: 5,
});
```

**Benefits:**
- **API Protection**: Prevents rate limit violations
- **Smooth Operation**: Consistent request flow
- **Resource Management**: Efficient bandwidth usage

### Connection Pooling

```javascript
const axios = Axios.create({
  timeout: env.HTTP_TIMEOUT,
  keepAlive: true,
  maxSockets: 10
});
```

**Benefits:**
- **Reduced Latency**: Reusable connections
- **Lower Overhead**: Fewer TCP handshakes
- **Better Performance**: Optimized resource usage

### Memory Management

```javascript
// Graceful shutdown
process.on('SIGINT', () => {
  logger.debug('Stopping...');
  newFeedWatcher.stop();
  process.exit(0);
});
```

**Benefits:**
- **Clean Shutdown**: Proper resource cleanup
- **Memory Leak Prevention**: Event listener cleanup
- **Graceful Termination**: Orderly process exit

## 📊 Monitoring & Observability

### Health Checks

```javascript
// Application startup validation
try {
  const bot = await parserService.getBot();
  const feedRoom = await parserService.getRoom(env.FEED_ROOM_ID);
  logger.info('Startup Complete!');
} catch (error) {
  logger.error('Startup failed');
  process.exit(2);
}
```

### Logging Destinations

**Console Logging:**
```
2024-01-15 10:30:45Z info [rss-service:app] RSS bot Loading, v1.0.0
2024-01-15 10:30:46Z info [rss-service:app] Bot Loaded: RSS Bot (rssbot@example.com)
2024-01-15 10:30:47Z info [rss-service:app] Feed Room Name: RSS Updates
```

**Syslog Integration:**
```javascript
new Syslog({
  host: env.SYSLOG_HOST,
  port: env.SYSLOG_PORT,
  protocol: env.SYSLOG_PROTOCOL,
  localhost: env.SYSLOG_SOURCE
});
```

**Loki Integration:**
```javascript
new LokiTransport({
  host: env.LOKI_HOST,
  labels: { app: appName }
});
```

## 🧪 Testing

### Local Testing

1. **Set up test environment**:
   ```bash
   cp .env.example .env
   # Edit .env with your values
   ```

2. **Use test RSS feed**:
   ```
   RSS_FEED_URL=https://lorem-rss.herokuapp.com/feed?unit=second&interval=30
   ```

3. **Enable debug logging**:
   ```
   CONSOLE_LEVEL=debug
   ```

4. **Run the application**:
   ```bash
   npm start
   ```

### Docker Testing

1. **Build local image**:
   ```bash
   docker build -t rss-bot-test .
   ```

2. **Run with environment variables**:
   ```bash
   docker run -e TOKEN=your_token -e FEED_ROOM_ID=your_room_id -e RSS_FEED_URL=your_feed_url rss-bot-test
   ```

3. **Use docker-compose for full testing**:
   ```bash
   docker-compose up
   ```

### Integration Testing

1. **Bot Validation**: Verify bot token and permissions
2. **Room Access**: Confirm bot membership in target room
3. **Feed Parsing**: Test RSS feed accessibility and parsing
4. **Message Posting**: Validate Webex message delivery
5. **Error Handling**: Test various failure scenarios

## 🔧 Customization

### Message Formatting

Customize the message format in `parserService.js`:

```javascript
function formatDescription(description) {
  // Custom formatting logic
  let formatted = description;
  formatted = formatted.replace(/\r?\n|\r/g, '<br />');
  formatted = formatted.replace(/<strong>-- /g, '<strong>');
  formatted = formatted.replace(/ --<\/strong>/g, '</strong>');
  return formatted;
}
```

### Feed Processing

Modify feed processing logic:

```javascript
async function parseFeed(item) {
  const output = {
    title: item.title,
    description: formatDescription(item.description),
    guid: item.guid,
    link: item.link,
    // Add custom fields
    author: item.author,
    category: item.category,
    pubDate: item.pubDate
  };
  
  // Custom HTML template
  const html = `
    <h4><a href="${output.link}">${output.title}</a></h4>
    <p><strong>Author:</strong> ${output.author}</p>
    <p>${output.description}</p>
    <p><small>Published: ${new Date(output.pubDate).toLocaleString()}</small></p>
  `;
  
  await httpService.postMessage(env.TOKEN, env.FEED_ROOM_ID, html);
}
```

### Multiple Feeds

Support multiple RSS feeds:

```javascript
const feeds = [
  { url: 'https://example.com/feed1.xml', roomId: 'room1' },
  { url: 'https://example.com/feed2.xml', roomId: 'room2' }
];

feeds.forEach(feed => {
  const watcher = new Watcher(feed.url, interval);
  watcher.on('new entries', (entries) => {
    entries.forEach(item => {
      parserService.parseFeed(item, feed.roomId);
    });
  });
  watcher.start();
});
```

## 🐛 Troubleshooting

### Common Issues

**Bot Not Receiving Messages:**
- Verify bot token is valid
- Check bot is added to target room
- Confirm room ID is correct
- Review bot permissions

**RSS Feed Not Parsing:**
- Validate RSS feed URL accessibility
- Check feed format compatibility
- Verify network connectivity
- Review proxy settings

**Docker Container Issues:**
- Check environment variables
- Verify image build success
- Review container logs
- Confirm port mappings

### Debug Commands

**View application logs:**
```bash
# Docker
docker logs rss-service -f

# Local
npm start
```

**Test RSS feed manually:**
```bash
curl -I "https://your-rss-feed-url.com/feed.xml"
```

**Validate Webex token:**
```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
     "https://webexapis.com/v1/people/me"
```

### Error Codes

| Code | Description | Solution |
|------|-------------|----------|
| 2 | Startup failure | Check token and room ID |
| 401 | Unauthorized | Verify bot token |
| 404 | Room not found | Check room ID and bot membership |
| 429 | Rate limited | Reduce polling frequency |

## 🚀 Production Deployment

### Docker Swarm

```yaml
version: '3.8'
services:
  rss-service:
    image: jeremywillans/rss-service:latest
    deploy:
      replicas: 1
      restart_policy:
        condition: on-failure
    environment:
      TOKEN: ${TOKEN}
      FEED_ROOM_ID: ${FEED_ROOM_ID}
      RSS_FEED_URL: ${RSS_FEED_URL}
    networks:
      - rss-network
```

### Kubernetes

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rss-service
spec:
  replicas: 1
  selector:
    matchLabels:
      app: rss-service
  template:
    metadata:
      labels:
        app: rss-service
    spec:
      containers:
      - name: rss-service
        image: jeremywillans/rss-service:latest
        env:
        - name: TOKEN
          valueFrom:
            secretKeyRef:
              name: rss-secrets
              key: token
        - name: FEED_ROOM_ID
          valueFrom:
            configMapKeyRef:
              name: rss-config
              key: room-id
        - name: RSS_FEED_URL
          valueFrom:
            configMapKeyRef:
              name: rss-config
              key: feed-url
```

### Environment-Specific Configuration

**Development:**
```env
CONSOLE_LEVEL=debug
RSS_INTERVAL=30
```

**Production:**
```env
CONSOLE_LEVEL=info
RSS_INTERVAL=300
LOKI_ENABLED=true
LOKI_HOST=http://loki:3100
```

## 🤝 Contributing

1. **Fork the repository**
2. **Create feature branch**: `git checkout -b feature/new-feature`
3. **Make changes** and add tests
4. **Follow code style**: `npm run format`
5. **Submit pull request** with detailed description

### Development Guidelines

- **Code Style**: Follow Airbnb JavaScript style guide
- **Testing**: Add unit tests for new features
- **Documentation**: Update README and inline comments
- **Security**: Follow security best practices

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🆘 Support

In case you've found a bug, please open a ticket with the [Webex Developer Support Team](https://developer.webex.com/support).

## ⚠️ Disclaimer

This application is provided as a sample only and is NOT guaranteed to be bug free and of Production quality.

## 🔗 Related Resources

- [Webex Bot Development Guide](https://developer.webex.com/docs/bots)
- [RSS 2.0 Specification](https://www.rssboard.org/rss-specification)
- [Docker Documentation](https://docs.docker.com/)
- [Node.js Best Practices](https://github.com/goldbergyoni/nodebestpractices)

## 🏷️ Tags

`webex` `rss` `bot` `nodejs` `docker` `automation` `integration` `messaging`

---

**Author**: Jeremy Willans  
**Repository**: https://github.com/WebexSamples/Boilerplate-RSS-bot
