<template>
  <div class="flex flex-col gap-10">
    <h1 class="text-3xl text-center">ZIP Code Search</h1>

    <form @submit.prevent="clear" class="flex justify-end">
      <button
        type="submit"
        class="w-48 h-12 border bg-violet-50 disabled:bg-white disabled:text-gray-200"
        :disabled="!hasResults && !hasGeo"
      >
        Clear
      </button>
    </form>

    <form @submit.prevent="searchZip" class="flex flex-col gap-5">
      <label for="zip">Enter ZIP Code: </label>
      <input
        type="text"
        id="zip"
        v-model="zipcode"
        placeholder="e.g. 12345"
        class="w-96 h-12 border text-2xl text-center"
      />
      <button type="submit" class="w-96 h-12 border bg-violet-50">
        Search
      </button>
    </form>
    <div v-if="zipLoading">Loading...</div>
    <div v-if="!zipLoading && hasResults">
      {{ result.city }}, {{ result.state }} {{ result.postal_code }}
    </div>
    <div v-if="!zipLoading && !hasResults">No results found</div>

    <form
      @submit.prevent="
        searchGeo();
        refUtm();
      "
      class="flex flex-col gap-5"
    >
      <button type="submit" class="w-96 h-12 border bg-violet-50">
        Your Geolocation
      </button>
    </form>
    <div class="flex flex-col w-96 min-h-[240px]">
      <div v-if="geoLoading">Loading...</div>
      <div v-if="!geoLoading && hasGeo">
        <div>{{ geo.city }}, {{ geo.regionName }} {{ geo.country }}</div>
        <div>{{ geo.isp }}</div>
        <div class="my-5">{{ referrer }}</div>
        <div v-for="(value, key) in utm" class="flex flex-col">
          {{ key }}: {{ value }}
        </div>
      </div>
      <div v-if="!geoLoading && !hasGeo">No geo found</div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      zipcode: "",
      zipLoading: false,
      result: null,
      ip: "",
      geoLoading: false,
      geo: null,
      referrer: "",
      utm: {},
    };
  },
  methods: {
    async searchZip() {
      if (!this.zipcode) return;
      this.zipLoading = true;
      try {
        const response = await fetch(
          `https://app.zipcodebase.com/api/v1/search?apikey=2d82ce80-c8bf-11ed-ab4a-af73f8651ed7&codes=${this.zipcode}&country=us`
        );
        const data = await response.json();
        if (this.zipcode in data.results) {
          this.result = data.results[this.zipcode][0];
          this.zipcode = "";
        } else {
          this.result = null;
        }
      } catch (error) {
        console.error(error);
      }
      this.zipLoading = false;
    },

    async searchGeo() {
      this.geoLoading = true;
      try {
        const response = await fetch("https://api.ipify.org?format=json");
        const data = await response.json();
        this.ip = data.ip ? data.ip : null;
      } catch (error) {
        console.error(error);
      }

      try {
        const response = await fetch(`http://ip-api.com/json/${this.ip}`);
        const data = await response.json();
        this.geo = data ? data : null;
      } catch (error) {
        console.error(error);
      }
      this.geoLoading = false;
    },

    async refUtm() {
      if (document.referrer) this.referrer = document.referrer;

      const self = window.location.toString();
      const querystring = self.split("?");
      if (querystring.length > 1) {
        const pairs = querystring[1].split("&");
        for (const i in pairs) {
          const key = pairs[i].split("=");
          if (key[0].startsWith("utm")) {
            this.utm[key[0]] = decodeURIComponent(key[1]);
          }
        }
      }
    },

    async clear() {
      this.result = "";
      this.geo = "";
    },
  },
  computed: {
    hasResults() {
      return !!this.result;
    },
    hasGeo() {
      return !!this.geo;
    },
  },
};
</script>
