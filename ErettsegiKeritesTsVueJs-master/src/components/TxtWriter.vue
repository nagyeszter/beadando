<template>
  <b-button id="myButton" block class="my-2" @click="writeTextToFile()">{{ title }}</b-button>
</template>

<script lang="ts">
import { Component, Prop, Vue } from "vue-property-decorator";

@Component
// A komponensek azonosítójában nem lehet ékezetes karakter pl.: TxtOlvasó !'
export default class TxtWriter extends Vue {
  @Prop() private title!: string;
  @Prop() private content!: string;
  @Prop() private filename!: string;

  public writeTextToFile(): void {
    if (!this.content) return;
    if (!this.filename) return;
    const textFileAsBlob = new Blob(["\ufeff", this.content], { type: "text/plain" });
    const downloadLink = document.createElement("a");
    downloadLink.download = this.filename;
    downloadLink.innerHTML = "Download File"; // Elhagyható?
    downloadLink.href = window.URL.createObjectURL(textFileAsBlob);
    downloadLink.onclick = e => document.body.removeChild(e.target as Node);
    downloadLink.style.display = "none";
    document.body.appendChild(downloadLink);
    downloadLink.click();
  }
}
</script>

<style>
#myButton {
  color: black;
  border: 0px;
  background-color: #bedce5;
  border-radius: 10px;
}
#myButton:hover {
  background-color: #daf4fd;
}
</style>
