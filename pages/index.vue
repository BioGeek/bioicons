<template>
  <div class="mx-auto relative" :class="{ dark: darkMode }">
    <section class="relative">
      <github-buttons />
      <div id="start" class="absolute top-0 right-0"></div>
      <app-header
        :dark-mode="darkMode"
        :numberoficons="icons.length"
        :counts="categoryCounts"
        @category="categorySelected"
      />
      <div class="absolute inset-x-0 top-full pointer-events-none">
        <svg
          preserveAspectRatio="none"
          fill="currentColor"
          viewBox="0 0 32 1"
          xmlns="http://www.w3.org/2000/svg"
          class="text-theme dark:text-dark-theme"
        >
          <path d="M16 1C4 1 0 0 0 0H32C32 0 28 1 16 1Z"></path>
        </svg>
      </div>
    </section>

    <section>
      <div
        class="px-0 lg:px-6 flex flex-row justify-center items-start"
        style="margin-top: calc(-1 * var(--search-bar-negative-margin))"
      >
        <main class="z-10 container mx-auto">
          <toolbar
            ref="toolbar"
            v-model="searchQuery"
            :dark-mode="darkMode"
            :clipboard="clipboard"
            :categories="categories"
            :clipboard-allowed="clipboardAllowed"
            :category="category"
            @category="categorySelected"
            @toggleDark="toggleDarkMode"
            @toggleClipboard="toggleClipboard"
          />
          <div
            class="shadow lg:shadow-lg bg-white dark:bg-cool-gray-900 overflow-hidden rounded-b-xl"
            style="
              margin-top: calc(-1 * var(--search-bar-height));
              padding-top: var(--search-bar-height);
              min-height: calc(100vh - 7rem);
            "
          >
            <!-- ClipboardAllowed {{ clipboardAllowed }} Clipboard {{ clipboard }} -->
            <div id="infiniteScroll">
              <div id="app-grid">
                <icon
                  v-for="(icon, i) in paginatedIcons"
                  :id="'icon' + i"
                  :key="icon.name"
                  :clipboard="clipboard"
                  :icon="icon"
                  @copy-clipboard="showToast"
                />
                <InfiniteScroll :enough="enough" @load-more="loadMore">
                  <!-- eslint-disable -->
                  <template> <loading class="sr-only" /></template>
                  <!-- eslint-enable -->
                </InfiniteScroll>
              </div>
            </div>
            <div
              v-if="filteredIcons.length == 0"
              class="flex flex-col items-center justify-center bg-contain h-screen bg-no-repeat bg-center"
              :style="noresults"
            >
              <span class="font-medium text-green-600 text-4xl mt-8"
                >No results</span
              >
              <span class="text-cool-gray-800 font-light mt-2">
                You can contribute your own by opening a
                <a
                  class="text-green-600 font-medium hover:underline focus:underline"
                  href="https://github.com/duerrsimon/bioicons"
                  rel="noopener noreferrer"
                  >new Github issue</a
                >
              </span>
            </div>
          </div>
        </main>
      </div>
      <div class="hidden lg:block h-24"></div>
    </section>
    <div class="container mx-auto mb-4 text-cool-gray-800 text-right">
      <button
        class="text-green-500 hover:underline font-medium"
        @click="displayTour()"
      >
        Show tour
      </button>
      &middot;
      <span>Made with Tailwind, Vue.js, Nuxt and Inkscape</span> &middot;
      <a
        href="https://github.com/duerrsimon/bioicons"
        class="text-green-500 hover:underline font-medium"
        rel="noopener noreferrer"
        >Contribute</a
      >
      your own icons by creating a pull request &middot;
      <a
        href="https://simonduerr.eu/impressum"
        class="text-green-500 hover:underline font-medium"
        rel="noopener noreferrer"
        >Imprint</a
      >
    </div>
    <notificationGroup>
      <div
        class="px-4 sm:px-6 py-4 fixed left-0 bottom-0 z-30 transition duration-300 ease-out opacity-100 transform scale-90"
      >
        <div class="rounded-1 shadow-4">
          <notification v-slot="{ notifications }">
            <div
              v-for="notification in notifications"
              :key="notification.id"
              class="rounded-1 shadow-2 px-3 py-2 bg-cool-gray-800 my-1"
            >
              <p
                class="font-medium text-cool-gray-100"
                style="font-size: 0.875rem; letter-spacing: 0.0125em"
              >
                <span class="flex flex-row"
                  ><span
                    class="flex flex-row items-center"
                    style="height: 1.3125rem"
                    ><svg
                      v-if="notification.type === 'copy'"
                      viewBox="0 0 20 20"
                      fill="currentColor"
                      class=""
                      style="width: 1em; height: 1em"
                    >
                      <path
                        fill-rule="evenodd"
                        d="M8 4a3 3 0 00-3 3v4a5 5 0 0010 0V7a1 1 0 112 0v4a7 7 0 11-14 0V7a5 5 0 0110 0v4a3 3 0 11-6 0V7a1 1 0 012 0v4a1 1 0 102 0V7a3 3 0 00-3-3z"
                        clip-rule="evenodd"
                      ></path>
                    </svg>
                    <svg
                      v-else
                      style="width: 1em; height: 1em"
                      fill="none"
                      stroke="currentColor"
                      viewBox="0 0 24 24"
                      xmlns="http://www.w3.org/2000/svg"
                    >
                      <path
                        stroke-linecap="round"
                        stroke-linejoin="round"
                        stroke-width="2"
                        d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"
                      ></path>
                    </svg> </span
                  ><span style="width: 1ch"></span
                  ><span v-if="notification.type === 'copy'"
                    >Copied {{ notification.text }} to clipboard</span
                  ><span v-else> {{ notification.text }} </span></span
                >
              </p>
            </div>
          </notification>
        </div>
      </div>
    </notificationGroup>
    <v-tour
      name="gettingStarted"
      :steps="steps"
      :options="{ startTimeout: 400 }"
      :callbacks="myCallbacks"
    ></v-tour>
  </div>
</template>

<script>
import noresult from 'assets/no-results.svg'
import AppHeader from '../components/AppHeader.vue'
import GithubButtons from '../components/GithubButtons.vue'
import Icon from '../components/Icon.vue'
import Toolbar from '../components/Toolbar.vue'
// import Loading from '../components/Loading.vue'

const getIcons = () =>
  import('../static/icons/icons.json').then((m) => m.default || m)

const getCategories = () =>
  import('../static/icons/categories.json').then((m) => m.default || m)

export default {
  components: { GithubButtons, AppHeader, Toolbar, Icon }, // Loading
  async asyncData({ req }) {
    const icons = await getIcons()
    const categories = await getCategories()

    let categoryCounts = {} // eslint-disable-line

    for (const { category } of icons) {
      categoryCounts[category] = categoryCounts[category]
        ? categoryCounts[category] + 1
        : 1
    }
    return { icons, categories, categoryCounts }
  },
  data() {
    return {
      noresults: { backgroundImage: `url(${noresult})` },
      myCallbacks: {
        onStop: this.onTourStop,
      },
      steps: [
        {
          target: '#start',
          header: {
            title: 'Hello 👋',
          },
          content: `Take a quick tour of bioicons.com`,
        },
        {
          target: '#v-step-0',
          header: {
            title: 'Categories',
          },
          content: `Bioicons are categorized so that you can more easily find them.`,
          params: {
            placement: 'top',
            enableScrolling: false,
          },
        },
        {
          target: '#clipboardDownload',
          header: {
            title: 'Clipboard/Download toggle',
          },
          content: `By clicking on this button, you can switch to direct download of icons as vector file. <br><br><small class="mb-2">If you are using recent Chrome/Edge browser, you can copy the icons on click to the clipboard an paste them. In Inkscape the image is pasted as png not as vector.</small> `,
        },
        {
          target: '#darkMode',
          header: {
            title: 'Darkmode toggle',
          },
          content: `Click this icon to view the site in dark mode.`,
        },
        {
          target: '#icon0 .license',
          header: {
            title: 'License',
          },
          content: `Before downloading check the license of the icon you are downloading.`,
          params: {
            placement: 'top',
            enableScrolling: false,
          },
        },
        {
          target: '#icon0 .attribution',
          header: {
            title: 'Attribution',
          },
          content: `For some licenses you need to attribute the creator for example in the figure caption or the acknowledgement section.`,
          params: {
            placement: 'top',
            enableScrolling: false,
          },
        },
      ],
      darkMode: false,
      clipboard: false,
      clipboardAllowed: true,
      // iconName: '',
      name: '',
      searchQuery: null,
      category: 'All_icons',
      loading: false,
      size: 12,
      enough: false,
      showTour: true,
    }
  },
  head() {
    return {
      title: 'Bioicons - high quality science illustrations',
      meta: [
        {
          hid: 'description',
          name: 'description',
          content:
            'Bioicons contains free science illustrations for biologists, chemists, machine learning under permissive CC0, CC BY or MIT License.',
        },
      ],
    }
  },
  computed: {
    categorizedIcons() {
      if (this.category !== 'All_icons') {
        return this.icons.filter((item) => {
          return item.category === this.category
        })
      } else {
        return this.icons
      }
    },
    settings() {
      return {
        clipboard: this.clipboard,
        darkMode: this.darkMode,
        showTour: this.showTour,
      }
    },
    filteredIcons() {
      if (this.searchQuery) {
        return this.categorizedIcons.filter((item) => {
          return this.searchQuery
            .toLowerCase()
            .split(' ')
            .every((v) => item.name.toLowerCase().includes(v))
        })
      } else {
        return this.categorizedIcons
      }
    },
    paginatedIcons() {
      return this.filteredIcons.slice(0, this.size)
    },
  },
  watch: {
    searchQuery() {
      this.$router.push({
        path: this.$route.path,
        query: { query: this.searchQuery },
      })
      this.size = 12
    },
    settings() {
      localStorage.setItem('settings', JSON.stringify(this.settings))
    },
  },
  created() {
    this.searchQuery = this.$route.query.query
  },
  mounted() {
    const settings = JSON.parse(localStorage.getItem('settings'))
    if (settings !== null) {
      if (typeof settings.clipboard !== 'undefined') {
        this.clipboard = settings.clipboard
      }
      if (typeof settings.darkMode !== 'undefined') {
        this.darkMode = settings.darkMode
      }
      if (typeof settings.showTour !== 'undefined') {
        this.showTour = settings.showTour
      }
      this.$notify(
        {
          text: 'Loaded settings from local storage',
          type: 'info',
        },
        2000
      )
    }
    if (
      this.$browserDetect.isFirefox ||
      this.$browserDetect.isIE ||
      this.$browserDetect.isSafari ||
      this.$browserDetect.isIOS ||
      this.$browserDetect.isChromeIOS
    ) {
      this.clipboard = true
      this.clipboardAllowed = false
    }
    window.addEventListener('keypress', (e) => {
      if (e.keyCode === 47) {
        e.preventDefault()
        this.$nextTick(() => {
          this.$refs.toolbar.$refs.searchInput.focus()
        })
      }
    })
    if (this.showTour) {
      this.$tours.gettingStarted.start()
    }
  },
  methods: {
    displayTour() {
      this.$tours.gettingStarted.start()
    },
    onTourStop() {
      this.showTour = false
    },
    categorySelected(val) {
      this.category = val
      this.size = 12
    },
    showToast(icon) {
      this.$notify(
        {
          text: icon,
          type: 'copy',
        },
        2000
      )
    },
    toggleDarkMode() {
      this.darkMode = !this.darkMode
    },
    toggleClipboard() {
      this.clipboard = !this.clipboard
    },
    loadMore() {
      this.size = this.size + 12
    },
  },
}
</script>

<style>
@import url('https://rsms.me/inter/inter.css');
html {
  font-family: 'Inter', sans-serif;
}
@supports (font-variation-settings: normal) {
  html {
    font-family: 'Inter var', sans-serif;
  }
}
@media (min-width: 1072px) {
  html {
    background-attachment: fixed, fixed;
    background-image: url("data:image/svg+xml,%3Csvg fill='rgba(16, 185, 129,50)' viewBox='0 0 1 1' xmlns='http://www.w3.org/2000/svg'%3E%3Crect width='1' height='1' /%3E%3C/svg%3E"),
      url("data:image/svg+xml,%3Csvg fill='rgba(16, 185, 129,50)' viewBox='0 0 32 1' xmlns='http://www.w3.org/2000/svg'%3E%3Cpath d='M16 1C4 1 0 0 0 0H32C32 0 28 1 16 1Z' /%3E%3C/svg%3E");
    background-repeat: repeat-x, no-repeat;
    background-size: 112px, 100%;
    background-position: 0 0, 0 112px;
  }
}
@media (min-width: 1072px) {
  html.dark {
    background-attachment: fixed, fixed;
    /* https://yoksel.github.io/url-encoder */
    background-image: url("data:image/svg+xml,%3Csvg fill='rgba(16, 185, 129,50)' viewBox='0 0 1 1' xmlns='http://www.w3.org/2000/svg'%3E%3Crect width='1' height='1' /%3E%3C/svg%3E"),
      url("data:image/svg+xml,%3Csvg fill='rgba(16, 185, 129,50)' viewBox='0 0 32 1' xmlns='http://www.w3.org/2000/svg'%3E%3Cpath d='M16 1C4 1 0 0 0 0H32C32 0 28 1 16 1Z' /%3E%3C/svg%3E");
    background-repeat: repeat-x, no-repeat;
    background-size: 112px, 100%;
    background-position: 0 0, 0 112px;
  }
}
#app-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(calc(300px - 1px), 1fr));
  gap: 1px;
}

#app-grid > * {
  outline: 1px solid #eee;
}

.dark #app-grid > * {
  outline: 1px solid #2d3748;
}
</style>
