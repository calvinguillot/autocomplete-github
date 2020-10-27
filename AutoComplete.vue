<template>
  <div class="autocomplete">
    <input
      class="auto-input"
      type="text"
      @input="search"
      @keydown.down="arrowDown"
      @keydown.up="arrowUp"
      @keydown.enter="enterKey"
      :placeholder="placeHolder"
    />
    <div id="auto-results" v-show="opened" class="auto-results">
      <div v-if="loading || items.length == 0" class="auto-item">
        {{ loading ? "Searching ..." : "No results." }}
      </div>
      <div
        v-else
        class="auto-item"
        :class="{ 'is-active': i === arrowPos }"
        v-for="(item, i) in sortResults"
        :key="i"
        @click="selectedItem(item)"
      >
        <div class="auto-item-group">
          <div class="auto-item-title">{{ item.title }}</div>
          <div class="auto-item-type">{{ item.type }}</div>
        </div>
        <div class="auto-item-url">{{ item.html_url }}</div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "AutoComplete2",
  data() {
    return {
      loading: false,
      debounce: null,
      opened: false,
      arrowPos: 0,
      items: [],
      minInit: 3,
      maxSearch: 50,
      sortBy: "title",
      placeHolder: "Search GitHub",
      serverTimeout: 3000,
    };
  },
  methods: {
    search(event) {
      if (event.target.value.length > this.minInit) {
        this.opened = true;
        this.loading = true;
        clearTimeout(this.debounce);
        this.debounce = setTimeout(() => {
          this.searchGit(event.target.value);
        }, this.serverTimeout);
      } else {
        this.opened = false;
      }
    },
    searchGit(query) {
      let users = this.queryType(query, "users", "User", ["login", "html_url"]);
      let repos = this.queryType(query, "repositories", "Repository", [
        "name",
        "html_url",
      ]);

      Promise.all([users, repos]).then((values) => {
        this.items = values[0].concat(values[1]);
        this.loading = false;
      });
    },
    queryType(input, search, type, properties) {
      const url = `https://api.github.com/search/${search}?q=${input}&per_page=${this.maxSearch}`;

      return new Promise((resolve) => {
        if (input.length < this.minInit) {
          return resolve([]);
        }

        fetch(url)
          .then((response) => response.json())
          .then((data) => {
            const results = [];

            data.items.forEach((e) => {
              const filtered = Object.keys(e)
                .filter((key) => properties.includes(key))
                .reduce((obj, key) => {
                  obj[key] = e[key];
                  return obj;
                }, {});
              filtered.type = type;
              filtered.title = filtered[properties[0]];

              results.push(filtered);
            });

            resolve(results);
          });
      });
    },
    selectedItem(result) {
      this.open = false;
      this.arrowCounter = -1;
      window.open(result.html_url, "_blank");
    },
    arrowDown() {
      if (this.arrowPos < this.items.length - 1) {
        this.arrowPos = this.arrowPos + 1;
        let active = document.getElementsByClassName("is-active");
        active[0].scrollIntoView();
      }
    },
    arrowUp() {
      if (this.arrowPos > 0) {
        this.arrowPos = this.arrowPos - 1;
        let active = document.getElementsByClassName("is-active");
        let parent = document.getElementById("auto-results");
        parent.scrollBy(0, -active[0].clientHeight);
      }
    },
    enterKey() {
      this.selectedItem(this.sortResults[this.arrowPos]);
    },
    clickOut(event) {
      if (!this.$el.contains(event.target)) {
        this.opened = false;
        this.arrowPos = -1;
      }
    },
  },
  computed: {
    sortResults() {
      let results = this.items;
      results.sort((a, b) => a[this.sortBy].localeCompare(b[this.sortBy]));
      return results;
    },
  },
  mounted() {
    document.addEventListener("click", this.clickOut);
  },
};
</script>

<style>
.autocomplete {
  width: 100%;
}
.auto-input {
  padding-left: 10px;
  width: 100%;
  height: 50px;
  border-radius: 5px;
  border: 2px solid #bdbdbd;
}

input:focus {
  outline: none;
  border-color: #3c60ff;
}

.auto-results {
  margin: 0;
  border: 1px solid #eeeeee;
  max-height: 250px;
  overflow: auto;
  width: 100%;
}

.auto-item {
  text-align: left;
  padding: 4px 12px;
  cursor: pointer;
  border-bottom: 1px solid #bdbdbd;
}

.auto-item-group {
  display: flex;
  margin-bottom: 10px;
}

.auto-item-title {
  flex: 1;
  font-weight: bold;
}
.auto-item-type {
  flex: 1;
  text-align: right;
}

.auto-item-url {
  word-wrap: break-word;
}

.auto-item.is-active,
.auto-item:hover {
  background-color: #3c60ff;
  color: white;
}
</style>