<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Proxy Search</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
        }
    </style>
</head>
<body class="bg-gray-100 flex justify-center items-center min-h-screen">
    <div class="bg-white p-8 rounded-lg shadow-md w-full max-w-md">
        <h1 class="text-2xl font-semibold text-blue-600 mb-4 text-center">Proxy Search</h1>
        <form id="search-form" class="space-y-4">
            <div>
                <label for="search-query" class="block text-gray-700 text-sm font-bold mb-2">Search</label>
                <input type="text" id="search-query" name="q" class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline" placeholder="Enter your search query..." required>
            </div>
            <div>
                <label for="proxy-server" class="block text-gray-700 text-sm font-bold mb-2">Proxy Server</label>
                <select id="proxy-server" name="proxy" class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline">
                    <option value="https://corsproxy.io/?">CorsProxy</option>
                    <option value="https://proxy.cors.sh/">CORS.SH</option>
                    <option value="https://api.allorigins.win/raw?url=">AllOrigins</option>
                    <option value="https://thingproxy.io/api/v1/?" disabled>ThingProxy (Disabled)</option>
                </select>
                <p class="text-gray-500 text-xs italic mt-1">Note: Some proxies may be down or blocked.  The default is CorsProxy.</p>
            </div>
            <input type="hidden" name="engine" value="gogoduck">
            <button type="submit" class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline w-full">Search with GoGoDuck</button>
        </form>
        <div id="results-container" class="mt-6">
            <h2 class="text-lg font-semibold text-gray-800 mb-3">Search Results</h2>
            <div id="results" class="bg-gray-50 border border-gray-200 rounded-md p-4">
                <p class="text-gray-600 italic">Loading...</p>
            </div>
        </div>
    </div>

    <script>
        const searchForm = document.getElementById('search-form');
        const resultsContainer = document.getElementById('results-container');
        const resultsDiv = document.getElementById('results');
        const proxySelect = document.getElementById('proxy-server');

        searchForm.addEventListener('submit', async (event) => {
            event.preventDefault();

            const searchQuery = document.getElementById('search-query').value;
            const proxy = proxySelect.value;
            const engine = searchForm.elements['engine'].value; // Hidden input value

            resultsDiv.innerHTML = '<p class="text-gray-600 italic">Loading...</p>';
            resultsContainer.style.display = 'block';

            let targetUrl = '';
            if (engine === 'gogoduck') {
                targetUrl = `https://api.gogoduck.com/search.php?q=${encodeURIComponent(searchQuery)}&format=json`;
            } else {
                targetUrl = `https://api.duckduckgo.com/search?q=${encodeURIComponent(searchQuery)}&format=json`;
            }


            const proxiedUrl = proxy + encodeURIComponent(targetUrl);


            try {
                const response = await fetch(proxiedUrl);

                if (!response.ok) {
                    throw new Error(`HTTP error! status: ${response.status}`);
                }
                const data = await response.json();

                if (engine === 'gogoduck')
                {
                    displayGoGoDuckResults(data);
                }
                else{
                    displayDuckDuckGoResults(data);
                }


            } catch (error) {
                console.error('Error fetching search results:', error);
                resultsDiv.innerHTML = `<p class="text-red-500">Error: ${error.message}.  Try a different proxy server.</p>`;
            }
        });

        function displayGoGoDuckResults(data) {
            if (!data || data.length === 0) {
                resultsDiv.innerHTML = '<p class="text-gray-600">No results found.</p>';
                return;
            }

            let resultsHTML = '<ul class="space-y-3">';
            data.forEach(item => {
                resultsHTML += `
                    <li class="bg-white rounded-md shadow-sm p-4">
                        <h2 class="text-lg font-semibold text-blue-700"><a href="${item.url}" target="_blank" rel="noopener noreferrer" class="hover:underline">${item.title}</a></h2>
                        <p class="text-gray-600">${item.snippet}</p>
                        <p class="text-sm text-gray-500"><a href="${item.url}" target="_blank" rel="noopener noreferrer" class="hover:underline">${item.url}</a></p>
                    </li>
                `;
            });
            resultsHTML += '</ul>';
            resultsDiv.innerHTML = resultsHTML;
        }

        function displayDuckDuckGoResults(data) {
            if (!data || !data.RelatedTopics || data.RelatedTopics.length === 0) {
                resultsDiv.innerHTML = '<p class="text-gray-600">No results found.</p>';
                return;
            }

            let resultsHTML = '<ul class="space-y-3">';
            data.RelatedTopics.forEach(item => {
                const text = item.Text || "";
                const url = item.FirstURL || "";

                resultsHTML += `
                    <li class="bg-white rounded-md shadow-sm p-4">
                        <h2 class="text-lg font-semibold text-blue-700"><a href="${url}" target="_blank" rel="noopener noreferrer" class="hover:underline">${text.split(' - ')[0]}</a></h2>
                        <p class="text-gray-600">${text}</p>
                        <p class="text-sm text-gray-500"><a href="${url}" target="_blank" rel="noopener noreferrer" class="hover:underline">${url}</a></p>
                    </li>
                `;
            });
            resultsHTML += '</ul>';
            resultsDiv.innerHTML = resultsHTML;
        }
    </script>
</body>
</html>
