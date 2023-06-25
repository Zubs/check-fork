<template>
  <v-card
    class="mx-auto mt-12"
    elevation="1"
    max-width="75%"
    :loading="loading"
  >
    <v-card-title class="py-5 font-weight-black">Find your forks easily</v-card-title>

    <v-card-text>
      To find a fork, simply paste the url of the original Github repository here.
    </v-card-text>

    <v-card-text>
      <div class="text-subtitle-2 font-weight-black mb-1">GitHub URL</div>

      <v-form @submit.prevent="search">
        <v-text-field
          label="https://github.com/username/repo"
          single-line
          variant="outlined"
          v-model="url"
          required
          clearable
          :rules="urlRules"
        ></v-text-field>

        <template v-if="errorMessage">
          <v-alert
            density="compact"
            type="error"
            title="Error Fetching Forks"
            :text="errorMessage"
          ></v-alert>
        </template>

        <v-btn
          :disabled="loading"
          :loading="loading"
          block
          class="text-none mb-4 mt-4"
          color="indigo-darken-3"
          size="x-large"
          variant="flat"
          prepend-icon="mdi-magnify"
          type="submit"
        >
          Search
        </v-btn>
      </v-form>

      <v-list v-if="forks.length" lines="two">
        <v-list-subheader inset>Forks: {{ forks.length }} of {{ totalCount }}</v-list-subheader>

        <v-list-item
          v-for="fork in forks"
          :key="fork.url"
          :title="fork.url.split('/').slice(-2).join('/')"
          :subtitle="'Last Updated At: ' + new Date(fork.updatedAt).toString().split(' ').slice(0, 4).join(' ')"
        >
          <template v-slot:prepend>
            <v-avatar :color="pink">
              <v-icon icon="mdi-github"></v-icon>
            </v-avatar>
          </template>

          <template v-slot:append>
            {{ fork.stargazerCount }}
            <v-btn
              color="grey-lighten-1"
              icon="mdi-star"
              variant="text"
            ></v-btn>
          </template>
        </v-list-item>
      </v-list>
    </v-card-text>
  </v-card>
</template>

<script>
export default {
  data: () => ({
    loading: false,
    url: '',
    urlRules: [
      v => !!v || 'URL is required',
      v => /^https:\/\/github.com\/[a-z0-9-]+\/[a-z0-9-]+$/i.test(v) || 'URL must be a valid GitHub repository',
    ],
    forks: [],
    totalCount: null,
    errorMessage: null,
  }),

  methods: {
    async search () {
      if (!this.url) return;

      this.loading = true;
      this.forks = [];
      this.totalCount = null;
      this.errorMessage = null;

      const data = await fetch('https://check-forks-f94a762b1338.herokuapp.com/search', {
        method: 'POST',
        body: JSON.stringify({ github_url: this.url }),
      }).then(res => res.json());

      if (data && data.success) {
        this.loading = false;
        this.forks = data.data.stars.data.repository.forks.nodes;
        this.totalCount = data.data.date.data.repository.forks.totalCount;
      } else {
        this.loading = false;
        this.errorMessage = data.message;
      }
    }
  },
}
</script>
