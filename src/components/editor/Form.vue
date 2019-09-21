<template>
  <div class="editor__column form-wrapper">
    <form class="form">
      <fieldset>
        <textarea @keyup="keyup" v-model="markdown"></textarea>
      </fieldset>
    </form>
  </div>
</template>

<style>
.form {
  position: relative;

  &,
  fieldset,
  textarea {
    height: 100%;
  }

  textarea {
    padding: 1em;
    width: 100%;
    resize: none;
    background-color: transparent;
    border: 0;
    color: #fff;
    font-family: monospace;
    font-size: 1.125em;
    line-height: 1.8;
  }
}
</style>

<script>
import Showdown from "showdown";
import Emoji from "emoji-js";

export default {
  name: "Form",
  data() {
    return {
      markdown: ""
    };
  },
  mounted() {
    if (this.previousStoredMarkdown)
      this.$data.markdown = this.previousStoredMarkdown;
    else this.initialState();
  },
  methods: {
    initialState() {
      const initialMarkdown = "# Start typing here";
      this.$data.markdown = initialMarkdown;

      this.$store.dispatch({
        type: "storeMarkdown",
        value: initialMarkdown
      });

      this.parse(initialMarkdown);
    },

    keyup() {
      const raw = this.rawMarkdown;

      this.parse(raw);

      this.$store.dispatch({
        type: "storeMarkdown",
        value: raw
      });
    },

    parse(raw) {
      // parse html
      const converter = new Showdown.Converter({
          simplifiedAutoLink: true,
          tasklists: true,
          tables: true,
          strikethrough: true
        }),
        parsedHTML = converter.makeHtml(raw);

      // parse emojis
      const emoji = new Emoji.EmojiConvertor(),
        htmlWithEmojis = emoji.replace_colons(parsedHTML);

      this.$store.dispatch({
        type: "storeHTML",
        value: htmlWithEmojis
      });
    }
  },
  computed: {
    rawMarkdown() {
      return this.$data.markdown;
    },

    previousStoredMarkdown() {
      return this.$store.getters["markdown"];
    }
  }
};
</script>