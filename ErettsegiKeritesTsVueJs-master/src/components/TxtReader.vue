<template>
  <b-form-group :label="label" label-for="myTxtFileLoader" label-cols-sm="3" class="m-2">
    <b-form-file
      id="myTxtFileLoader"
      v-model="file"
      accept=".txt"
      :state="Boolean(file)"
      :placeholder="placeholder"
      browse-text="Keresés..."
      drop-placeholder="Dobja ide!"
      class="shadow"
    ></b-form-file>
  </b-form-group>
</template>

<script lang="ts">
import { Component, Prop, Watch, Vue } from "vue-property-decorator";

@Component
// A komponensek azonosítójában nem lehet ékezetes karakter pl.: TxtOlvasó !
export default class TxtReader extends Vue {
  private file: any = null;
  @Prop() private label!: string;
  @Prop() private placeholder!: string;

  @Watch("file", { immediate: true, deep: true })
  private haFileValtozik() {
    if (this.file) {
      const reader: FileReader = new FileReader();
      reader.onload = e => {
        if (e.target) {
          this.$emit("load", reader.result);
        }
      };
      if (this.file) {
        reader.readAsText(this.file);
      }
    }
  }
}
</script>

<style></style>
