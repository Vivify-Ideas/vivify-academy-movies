<template>
  <div class="container">
    <h1>{{ magicNumber }}</h1>

    <div class="pt-3">
      <b-badge
        pill
        variant="primary"
        v-if="movies.length"
      >

        Selected: {{ selectedMoviesCounter }}
      </b-badge>

        <b-dropdown
          text="Sort By"
          class="m-md-2"
          size="sm"
        >
          <b-dropdown-item
            @click="sortBy('title', 'asc')"
          >
            Name Asc
          </b-dropdown-item>
          <b-dropdown-item
            @click="sortBy('title', 'desc')"
          >
          Name Desc
          </b-dropdown-item>
          <b-dropdown-divider></b-dropdown-divider>
          <b-dropdown-item
            @click="sortBy('duration', 'asc')"
          >
            Duration Asc
          </b-dropdown-item>
          <b-dropdown-item
            @click="sortBy('duration', 'desc')"
          >
          Duration Desc
          </b-dropdown-item>
        </b-dropdown>

      <button
        type="button"
        class="btn btn-primary btn-sm"
        @click="selectAllMovies">
        Select all
      </button>
      <button
        type="button"
        class="btn btn-warning btn-sm"
        @click="deselectMovies"
      >
        Deselect all
      </button>

      <movie-row
        v-for="movie in currentlyVisibleMovies"
        :key="movie.id"
        :movie="movie"
        :selected-movies-ids="selectedMoviesIds"
        @on-selected-movie="onSelectedMovie"
      />

      <movies-paginator
        :pages="pages"
        :selected-page="selectedPage"
        v-if="pages > 1"
        @selected-page="onSelectedPage"
      />

      <b-alert
        show
        variant="warning"
        v-if="!movies.length"
      >
        No Movies
      </b-alert>
    </div>
  </div>
</template>

<script>
import MoviesService from './../services/MoviesService'
import MovieRow from './MovieRow.vue'
import MovieSearch from './MovieSearch.vue'
import MovieForm from './MovieForm.vue'
import MoviesPaginator from './MoviesPaginator.vue'


import { mapGetters, mapMutations, mapActions } from 'vuex'

export default {
  name: 'AppMovies',
  components: {
    MovieRow,
    MovieSearch,
    MovieForm,
    MoviesPaginator
  },
  data() {
    return {
      // movies: [],
      selectedMoviesIds: [],
      selectedPage: 1,
      // currentTerm: '',
      intervalId: null
    }
  },
  // [ 1, 2, 3] => ...[ 1, 2, 3 ] => 1, 2, 3
  computed: {
    ...mapGetters({
      magicNumber: 'getCounter',
      currentTerm: 'getSearchTerm',
      movies: 'getMovies'
    }),
    selectedMoviesCounter() {
      return this.selectedMoviesIds.length
    },
    pages() {
      return Math.ceil(this.movies.length / 5)
    },
    filteredMovies() {
      return this.movies.filter((movie) => {
        return movie.title.toLowerCase()
          .indexOf(
            this.currentTerm.toLowerCase()
          ) > -1
      })
    },
    currentlyVisibleMovies() {
      let bottomLimit = (this.selectedPage - 1) * 5
      let topLimit = bottomLimit + 5
      return this.filteredMovies.filter((movie, index) => {
        return index >= bottomLimit && index < topLimit
      })
    }
  },
  methods: {
    ...mapMutations([
      'incrementCounter'
    ]),
    ...mapActions([
      'incrementCounterAction',
      'fetchMovies'
    ]),
    storeMovie() {
      MoviesService.store(this.movieForm)
    },
    onSearchTermChanged(term) {
      this.currentTerm = term
      // MoviesService.index(term)
      //   .then(({ data }) => {
      //     this.movies = data
      //   })
    },
    onSelectedMovie(movie, isSelected) {
      if (!isSelected) {
        let movieIndex = this.selectedMoviesIds.indexOf(movie.id);
        this.selectedMoviesIds.splice(movieIndex, 1);
        return;
      }
      this.selectedMoviesIds.push(movie.id)
    },
    deselectMovies() {
      this.selectedMoviesIds = [];
    },
    selectAllMovies() {
      this.selectedMoviesIds =
        this.movies.map((movie) => movie.id)
    },
    sortBy(prop, order) {
      let orderCoefficient = order === 'asc' ? 1 : -1;
      // console.log(prop, order);
      this.movies = this.movies.sort((movie1, movie2) => {
        return movie1[prop] >= movie2[prop] ?
          orderCoefficient : -orderCoefficient;
      });
    },
    onSelectedPage(page) {
      this.selectedPage = page
    }
  },
  // beforeRouteEnter(to, from, next) {
  //   // MoviesService.index().then(({ data }) => {
  //   //   next((context) => {
  //   //     context.movies = data.map((movie) => {
  //   //       movie.duration = parseInt(movie.duration)
  //   //       return movie
  //   //     });
  //   //   })
  //   // })
  // },
  mounted() {
    this.fetchMovies()
    this.incrementCounterAction()
  }
}
</script>
