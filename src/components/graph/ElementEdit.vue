<template>
  <v-card class="element-edit-card">
    <v-system-bar
        class="drag-handle"
        color="grey"
        dark
    >
      <v-spacer></v-spacer>
      <v-icon @click="handleCloseClick">mdi-close</v-icon>
    </v-system-bar>
    <v-form v-on:drag.prevent.self>
      <v-container>
        <v-row>
          <v-col
              cols="12"
              sm="12"
              md="12"
          >
            <v-text-field label="Title" v-model="label"></v-text-field>
            <v-text-field label="Fill Color" v-model="bodyFill"></v-text-field>
            <v-text-field label="Stroke Color" v-model="bodyStroke"></v-text-field>
          </v-col>
        </v-row>
      </v-container>
    </v-form>
  </v-card>
</template>

<script type="ts">
export default {
  props: {
    element: Object
  },
  computed: {
    label: {
      get () {
        return this.element.attr('label/text');
      },
      set (val) {
        this.element.attr('label/text', val);
      }
    },
    bodyFill: {
      get () {
        return this.element.attr('body/fill');
      },
      set (val) {
        this.element.attr('body/fill', val);
      }
    },
    bodyStroke: {
      get () {
        return this.element.attr('body/stroke');
      },
      set (val) {
        this.element.attr('body/stroke', val);
      }
    },
  },
  methods: {
    handleCloseClick () {
      console.log("Click");
      this.$emit('close');
    }
  }
}
</script>

<style type="scss">
.drag-handle:hover {
  cursor: move;
}
.close-button {
  position: absolute;
  top: 0px;
  right: 0px;
  background: white;
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