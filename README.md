# Oracle HCM Candidate Experience Proxy (Node/Express)

This repository contains a minimal Node.js/Express proxy that forwards all incoming requests to an Oracle HCM Cloud Candidate Experience endpoint. It preserves response status and content-type, injects the `ora-irc-vanity-domain` header, and rewrites redirect `Location` headers so that they point to a vanity domain.

- **Tech stack**: Node.js, Express, node-fetch
- **Entry point**: `index.js`
- **Start script**: `npm start`

## How it works

- **Base target**: Requests are proxied to `https://egue-dev12.fa.us2.oraclecloud.com/hcmUI/CandidateExperience`.
- **Routing**: Every incoming path and query string are appended to the base target.
- **Header injection**: Adds `ora-irc-vanity-domain: Y` to the upstream request.
- **Redirect handling**: Upstream redirects are not auto-followed. If a response includes `Location` and it starts with the base target URL, the proxy rewrites it to the vanity domain `https://orcdemo.work` but preserves the original path and query.
- **Response passthrough**: Returns upstream status, `Content-Type`, and body to the client.
- **Logging**: Logs incoming requests, the target proxy URL, and any final redirect location.

## Important limitations

- **HTTP method and body**: All upstream requests are currently sent as `GET` without forwarding client headers or body. If you need to support other methods (POST/PUT/DELETE) or forward payloads/headers, you will need to extend `index.js` accordingly.

## Local development

### Prerequisites
- Node.js 18+ recommended
- npm 9+

### Install and run
```bash
npm install
npm start
```
The server listens on port `8080` by default or on the port specified via the `PORT` environment variable.

### Example request
```bash
curl -i "http://localhost:8080/en/sites/CX_1?keyword=engineer"
```
This proxies to:
```
https://egue-dev12.fa.us2.oraclecloud.com/hcmUI/CandidateExperience/en/sites/CX_1?keyword=engineer
```

## Configuration

The following values are defined in `index.js`:

- **PORT**: Port to listen on. Default: `8080`. Override via environment variable `PORT`.
- **BASE_URL**: Upstream Oracle HCM Candidate Experience base URL. Default (hardcoded):
  `https://egue-dev12.fa.us2.oraclecloud.com/hcmUI/CandidateExperience`
- **Vanity domain for redirects**: When the upstream responds with a redirect whose `Location` starts with `BASE_URL`, the proxy rewrites it to use `https://orcdemo.work` while preserving the path and query.

If you need to change `BASE_URL` or the vanity domain, edit those constants in `index.js`.

## Deployment

### Azure App Service (Windows/IISNode)
This repo includes `web.config` for IISNode. On Windows-based Azure App Services, it rewrites all requests to `index.js` so that Express can handle them.

### Linux containers or generic hosts
The `web.config` is not used. Run the app using the start script:
```bash
npm ci
npm start
```
Ensure the `PORT` environment variable is set by your platform (or keep the default 8080) and expose that port.

## Project scripts

- **start**: `node index.js`

## Dependencies

- `express`: HTTP server framework
- `node-fetch`: Used to proxy upstream requests and control redirect handling

## Troubleshooting

- Ensure outbound network access from your host to the Oracle Cloud domain is allowed.
- Check logs for the target URL and redirect location to diagnose issues.
- If you see unexpected redirect behavior, verify the `BASE_URL` and the vanity domain values in `index.js`.

## License

ISC (see `package.json`).