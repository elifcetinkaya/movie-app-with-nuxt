<template>
  <div class="home">
    <!--Hero-->
    <HeroComponent />

    <!--Search-->
    <div class="container search">
      <input
        v-model.lazy="searchInput"
        type="text"
        placeholder="Search"
        @keyup.enter="$fetch"
      />
      <button v-show="searchInput !== ''" class="button" @click="clearSearch">
        Clear Search
      </button>
    </div>

    <!--WatchList Button-->
    <div class="container">
      <NuxtLink class="button watch button-light" :to="{ name: 'watchlist' }">
        Watch List
      </NuxtLink>
    </div>

    <!--Loading-->
    <LoadingComponent v-if="$fetchState.pending" />

    <!--Movies-->
    <div v-else class="container movies">
      <div v-if="searchInput !== ''" id="movie-grid" class="movies-grid">
        <div
          v-for="(movie, index) in searchedMovies"
          :key="index"
          class="movie"
        >
          <div class="movie-img">
            <img
              :src="`https://image.tmdb.org/t/p/w500/${movie.poster_path}`"
              alt=""
            />
            <p class="review">{{ movie.vote_average }}</p>
            <p class="overview">{{ movie.overview }}</p>
          </div>
          <div class="info">
            <p class="title">
              {{ movie.title.slice(0, 25)
              }}<span v-if="movie.title.length > 25">...</span>
            </p>
            <p class="release">
              Released:
              {{
                new Date(movie.release_date).toLocaleString('en-us', {
                  month: 'long',
                  day: 'numeric',
                  year: 'numeric',
                })
              }}
            </p>
            <NuxtLink
              class="button button-light"
              :to="{ name: 'movies-id', params: { id: movie.id } }"
            >
              Get More Info
            </NuxtLink>
            <button class="button button-light">Add to Watch List</button>
          </div>
        </div>
      </div>

      <div v-else id="movie-grid" class="movies-grid">
        <div v-for="(movie, index) in movies" :key="index" class="movie">
          <div class="movie-img">
            <img
              :src="`https://image.tmdb.org/t/p/w500/${movie.poster_path}`"
              alt=""
            />
            <p class="review">{{ movie.vote_average }}</p>
            <p class="overview">{{ movie.overview }}</p>
          </div>
          <div class="info">
            <p class="title">
              {{ movie.title.slice(0, 25)
              }}<span v-if="movie.title.length > 25">...</span>
            </p>
            <p class="release">
              Released:
              {{
                new Date(movie.release_date).toLocaleString('en-us', {
                  month: 'long',
                  day: 'numeric',
                  year: 'numeric',
                })
              }}
            </p>
            <NuxtLink
              class="button button-light"
              :to="{ name: 'movies-movieid', params: { id: movie.id } }"
            >
              Get More Info
            </NuxtLink>
            <button
              class="button button-light"
              :disabled="checkIsAdded(movie.id)"
              @click="addWatchList(movie.id)"
            >
              <span v-if="checkIsAdded(movie.id)">Added</span>
              <span v-else> Add to Watch List </span>
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  data() {
    return {
      movies: [],
      searchedMovies: [],
      searchInput: '',
      watchList: [],
    }
  },

  async fetch() {
    if (this.searchInput === '') {
      await this.getMovies()
    }
    await this.getWatchList()
  },

  fetchDelay: 100,

  methods: {
    async getMovies() {
      const data = axios.get(
        `https://api.themoviedb.org/3/movie/now_playing?api_key=37ed43a4f8eaa2abd75f9283692947bc&language=en-US&page=1`
      )
      const result = await data
      result.data.results.forEach((movie) => {
        this.movies.push(movie)
      })
    },

    async searchMovies() {
      const data = await axios.get(
        `https://api.themoviedb.org/3/search/movie?api_key=37ed43a4f8eaa2abd75f9283692947bc&language=en-US&page=1&query=${this.searchInput}`
      )
      const result = await data
      result.data.results.forEach((movie) => {
        this.searchedMovies.push(movie)
      })
    },

    async addWatchList(id) {
      const data = axios.post(
        'https://api.themoviedb.org/3/account/10557011/watchlist',
        {
          media_type: 'movie',
          media_id: id,
          watchlist: true,
        },
        {
          headers: {
            accept: 'application/json',
            'content-type': 'application/json',
            Authorization:
              'Bearer eyJhbGciOiJIUzI1NiJ9.eyJhdWQiOiJmNDhjMWJhM2FkN2Y3ZTM1OTFlYjM3YjViMDJhNjcyMSIsInN1YiI6IjYwYjBiMGMwYzVjMWVmMDA1OWU3YWE5ZiIsInNjb3BlcyI6WyJhcGlfcmVhZCJdLCJ2ZXJzaW9uIjoxfQ.FjOlMRnhzN533v0F8OuGfEynaMra55OeU6C3-WArzuU',
          },
        }
      )
      await data
      await this.getWatchList();
    },

    async getWatchList() {
      const data = axios.get(
        'https://api.themoviedb.org/3/account/10557011/watchlist/movies?language=en-US&page=1&sort_by=created_at.asc',
        {
          headers: {
            accept: 'application/json',
            'content-type': 'application/json',
            Authorization:
              'Bearer eyJhbGciOiJIUzI1NiJ9.eyJhdWQiOiJmNDhjMWJhM2FkN2Y3ZTM1OTFlYjM3YjViMDJhNjcyMSIsInN1YiI6IjYwYjBiMGMwYzVjMWVmMDA1OWU3YWE5ZiIsInNjb3BlcyI6WyJhcGlfcmVhZCJdLCJ2ZXJzaW9uIjoxfQ.FjOlMRnhzN533v0F8OuGfEynaMra55OeU6C3-WArzuU',
          },
        }
      )
      const result = await data
      result.data.results.forEach(movie => {
        this.watchList.push(movie)
      })
    },

    clearSearch() {
      this.searchInput = ''
      this.searchedMovies = []
    },

    checkIsAdded(id) {
      return this.watchList.find(movie => movie.id === id)
    },
  },
}
</script>

<style scoped lang="scss">
.home {
  .loading {
    padding-top: 120px;
    align-items: flex-start;
  }

  .search {
    display: flex;
    padding: 32px 16px;

    input {
      max-width: 350px;
      width: 100%;
      padding: 12px 6px;
      font-size: 14px;
      border: none;

      &:focus {
        outline: none;
      }
    }

    .button {
      border-top-left-radius: 0;
      border-bottom-left-radius: 0;
    }
  }

  .movies {
    padding: 32px 16px;
    .movies-grid {
      display: grid;
      column-gap: 32px;
      row-gap: 64px;
      grid-template-columns: 1fr;
      @media (min-width: 500px) {
        grid-template-columns: repeat(2, 1fr);
      }
      @media (min-width: 750px) {
        grid-template-columns: repeat(3, 1fr);
      }
      @media (min-width: 1100px) {
        grid-template-columns: repeat(4, 1fr);
      }

      .movie {
        position: relative;
        display: flex;
        flex-direction: column;

        .movie-img {
          position: relative;
          overflow: hidden;

          &:hover {
            .overview {
              transform: translateY(0);
            }
          }

          img {
            display: block;
            width: 100%;
            height: 100%;
          }

          .review {
            position: absolute;
            top: 0;
            left: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            width: 40px;
            height: 40px;
            background-color: #c92502;
            color: #fff;
            border-radius: 0 0 16px 0;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1),
              0 2px 4px -1px rgba(0, 0, 0, 0.06);
          }

          .overview {
            line-height: 1.5;
            position: absolute;
            bottom: 0;
            background-color: rgba(201, 38, 2, 0.9);
            padding: 12px;
            color: #fff;
            transform: translateY(100%);
            transition: 0.3s ease-in-out all;
          }
        }

        .info {
          margin-top: auto;
          .title {
            margin-top: 8px;
            color: #fff;
            font-size: 20px;
          }

          .release {
            margin-top: 8px;
            color: #c9c9c9;
          }

          .button {
            margin-top: 8px;
          }
        }
      }
    }
  }

  .watch {
    margin-left: 24px;
  }
}
</style>
