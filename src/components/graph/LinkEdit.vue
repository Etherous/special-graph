<template>
  <v-card class="link-edit-card">
    <v-system-bar
        class="drag-handle"
        color="grey"
        dark
    >
      <v-spacer></v-spacer>
      <v-icon @click="handleCloseClick">mdi-close</v-icon>
    </v-system-bar>
    <v-form>
      <v-container>
        <v-row>
          <v-col
              cols="12"
              sm="12"
              md="12"
          >
            <v-text-field class="edit-text" label="Title" v-model="label"></v-text-field>
            <v-text-field class="edit-text" label="Stroke Color" v-model="strokeColor"></v-text-field>
          </v-col>
        </v-row>
      </v-container>
    </v-form>
  </v-card>
</template>

<script type="ts">
export default {
  props: {
    link: Object
  },
  computed: {
    label: {
      get() {
        return this.link.label(0)?.attrs.text.text;
      },
      set(val) {
        this.link.label(0,{
          attrs: {
            text: {
              text: val
            }
          }
        });
      }
    },
    strokeColor: {
      get() {
        return this.link.attr('line/stroke');
      },
      set(val) {
        this.link.attr('line/stroke', val);
      }
    },

  },
  methods: {
    handleCloseClick() {
      this.$emit('close');
    }
  }
}
</script>

<style type="scss">
.close-button {
  position: absolute;
  top: 0px;
  right: 0px;
  /*width: 1em;*/
  /*height: 1em;*/
  /*text-align: center;*/
  /*border-radius: 0px;*/
}

.close-button:hover {
  cursor: pointer;
}

.close-x {
  top: -2px;
  /*font-family: monospace;*/
  /*font-weight: bold;*/
  /*line-height: 1em;*/
  /*color: white;*/
}
</style>
