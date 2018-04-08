<template>

  <div class="column">
    <Header :title="title" :items="tabItems" :actions="actions"/>
    <hr>
    <TabList :items="tabItems"/>
    <p v-if="tabItems.length == 0">{{ fallbackText }}</p>
    <InputCard/>
  </div>

</template>

<script>

import Header from "./Header.vue";
import TabList from "./TabList.vue";
import InputCard from "./InputCard.vue";

export default {
  components: {
    Header,
    TabList,
    InputCard
  },
  data: function() {
    return {
      title: "Tabs",
      tabItems: [],
      actions: {
        "Save": {
          handle: this.handleSave,
          disabled: true
        }
      },
      fallbackText: "Nothing here. Try opening a few tabs."
    }
  },
  created: function() {
    // Reactive tabs
    var vue = this;
    vue.updateTabItems();
    browser.tabs.onUpdated.addListener(function() {
      vue.updateTabItems();
    });
    browser.tabs.onRemoved.addListener(function() {
      setTimeout(vue.updateTabItems, 500);
    });
    // Hotkey
    window.addEventListener("keydown", function(event) {
      // Space
      if (event.keyCode == 32) {
        vue.handleSave();
      }
    });
  },
  methods: {
    updateTabItems: function() {
      // Update tab list with new items
      var vue = this;
      browser.tabs.query({
        currentWindow: true
      }).then(function(tabs) {
        var openTabs = tabs.filter(tab =>
          tab.url.startsWith("http") || tab.url.startsWith("https")
        );
        var tabList = openTabs.map(function(tab) {
          return {
            favicon: tab.favIconUrl,
            title: tab.title,
            url: tab.url
          };
        });
        vue.tabItems = tabList;
        // Enable or disable save action
        if (tabList.length == 0) {
          vue.actions["Save"].disabled = true;
        } else {
          vue.actions["Save"].disabled = false;
        }
      });
    },
    handleSave: function() {
      // Show input card
      if (!this.actions["Save"].disabled) {
        card.classList.add("is-active");
        cardInput.value = "";
        cardInput.focus();
      }
    }
  }
}

</script>