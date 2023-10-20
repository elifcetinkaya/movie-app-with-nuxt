<template>
  <div class="home">
    <!--Hero-->

    <!--Search-->
    <div class="container search">
      <input
        v-model.lazy="searchInput"
        type="text"
        placeholder="Search"
        @keyup.enter="searchMovies"
      />
      <button v-show="searchInput !== ''" class="button" @click="clearSearch">
        Clear Search
      </button>
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
            <button class="button button-light" @click="deleteMovie(movie.id)">
              Delete
            </button>
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
            <button class="button button-light" @click="deleteMovie(movie.id)">
              Delete
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
    }
  },

  async fetch() {
    await this.getWatchList()
  },

  onMounted() {
    this.getWatchList()
  },

  methods: {
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
      result.data.results.forEach((movie) => {
        this.movies.push(movie)
      })
    },

    async deleteMovie(id) {
      const data = axios.post(
        'https://api.themoviedb.org/3/account/10557011/watchlist',
        {
          media_type: 'movie',
          media_id: id,
          watchlist: false,
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
      this.movies = this.movies.filter((movie) => movie.id !== id)
    },

    searchMovies() {
      if (this.searchInput.trim() === '') {
        this.searchedMovies = this.movies
      } else {
        const keyword = this.searchInput.trim().toLowerCase()
        this.searchedMovies = this.movies.filter((movie) => {
          return movie.title.toLowerCase().includes(keyword)
        })
      }
    },

    clearSearch() {
      this.searchInput = ''
      this.searchedMovies = []
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
