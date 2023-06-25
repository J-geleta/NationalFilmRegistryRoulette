// Import necessary modules
const express = require('express');
const axios = require('axios');
const cheerio = require('cheerio');

// Initialize Express app
const app = express();
const port = process.env.PORT || 3000;

// Serve static files from the public directory
app.use(express.static('public'));

// Import node-fetch and assign it to a fetch variable
let fetch;
import('node-fetch').then(nodeFetch => {
    fetch = nodeFetch.default;

    // Define the route handler for the '/movies' endpoint
    app.get('/movies', async (req, res) => {

        // Fetch the National Film Registry Wikipedia page
        const wikiResponse = await axios.get('https://en.wikipedia.org/wiki/National_Film_Registry');

        // Load the HTML data into Cheerio
        const $ = cheerio.load(wikiResponse.data);

        // Extract the movie titles from the page
        const movies = [];
        $('table.wikitable').first().find('i').each((index, element) => {
            const movieTitle = $(element).text();
            movies.push({title: movieTitle});
        });

        // Randomly select a movie from the list
        const selectedMovie = movies[Math.floor(Math.random() * movies.length)];

        // Use TMDB API to search for details of the selected movie
        const tmdbSearchResponse = await fetch(`https://api.themoviedb.org/3/search/movie?api_key=1568b001d1f02653e899ed311409366d&query=${encodeURIComponent(selectedMovie.title)}`);
        const tmdbSearchData = await tmdbSearchResponse.json();
        const tmdbMovie = tmdbSearchData.results[0];

        // If the movie is found in TMDB, fetch additional details
        if (tmdbMovie) {

            // Fetch watch provider information
            const tmdbProviderResponse = await fetch(`https://api.themoviedb.org/3/movie/${tmdbMovie.id}/watch/providers?api_key=1568b001d1f02653e899ed311409366d`);
            const tmdbProviderData = await tmdbProviderResponse.json();
            selectedMovie.providers = tmdbProviderData.results.US;

            // Fetch movie details such as poster, tagline, overview, and release date
            const tmdbDetailResponse = await fetch(`https://api.themoviedb.org/3/movie/${tmdbMovie.id}?api_key=1568b001d1f02653e899ed311409366d`);
            const tmdbDetailData = await tmdbDetailResponse.json();
            selectedMovie.poster_path = tmdbDetailData.poster_path;
            selectedMovie.tagline = tmdbDetailData.tagline || "Tagline not available.";
            selectedMovie.overview = tmdbDetailData.overview;
            selectedMovie.release_date = tmdbDetailData.release_date;

            // Extract release year from release date
            if (tmdbDetailData.release_date) {
                selectedMovie.year = tmdbDetailData.release_date.split('-')[0];
            }

            // Fetch movie credits to extract the director's name
            const tmdbCreditsResponse = await fetch(`https://api.themoviedb.org/3/movie/${tmdbMovie.id}/credits?api_key=1568b001d1f02653e899ed311409366d`);
            const tmdbCreditsData = await tmdbCreditsResponse.json();
            const director = tmdbCreditsData.crew.find(person => person.job === 'Director');
            selectedMovie.director = director ? director.name : 'Unknown';
        }

        // Send the movie information back as JSON
        res.json(selectedMovie);

    });

    // Start the server
    app.listen(port, () => {
        console.log(`Server running at http://localhost:${port}`);
    });
});
