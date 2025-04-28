# Proxy Search Engine Using Gogoduck

const http = require('http');
const https = require('https');
const url = require('url');

const PORT = 3000;

const requestHandler = (req, res) => {
    const parsedUrl = url.parse(req.url, true);
    const searchQuery = parsedUrl.query.q;

    if (!searchQuery) {
        res.writeHead(400, { 'Content-Type': 'text/plain' });
        res.end('Query parameter "q" is required.');
        return;
    }

    const gogoduckUrl = `https://gogoduck.com/search?q=${encodeURIComponent(searchQuery)}`;

    https.get(gogoduckUrl, (gogoduckRes) => {
        let data = '';

        gogoduckRes.on('data', (chunk) => {
            data += chunk;
        });

        gogoduckRes.on('end', () => {
            res.writeHead(200, { 'Content-Type': 'text/html' });
            res.end(data);
        });
    }).on('error', (err) => {
        res.writeHead(500, { 'Content-Type': 'text/plain' });
        res.end('Error fetching data from Gogoduck.');
    });
};

const server = http.createServer(requestHandler);

server.listen(PORT, () => {
    console.log(`Proxy server is running on http://localhost:${PORT}`);
});
