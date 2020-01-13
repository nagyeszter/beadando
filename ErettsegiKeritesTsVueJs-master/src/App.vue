<template>
  <div id="app">
    <b-navbar id="myNavbar" toggleable="lg">
      <b-navbar-brand>Kerítés feladat</b-navbar-brand>
      <b-navbar-toggle target="nav-collapse"></b-navbar-toggle>
      <b-collapse id="nav-collapse" is-nav>
        <b-navbar-nav>
          <b-nav-item class="shadow-sm m-1 rounded myNavitem" href="kerites.txt" :link-attrs="downloadPropObj"
            >kerites.txt forrás letöltése</b-nav-item
          >
          <b-nav-item class="shadow-sm m-1 rounded myNavitem" href="Kerítés_fel.pdf" target="_blank">Feladat</b-nav-item>
          <b-nav-item class="shadow-sm m-1 rounded myNavitem" href="Kerítés_jav.pdf" target="_blank">Javítási útmutató</b-nav-item>
          <b-nav-item class="shadow-sm m-1 rounded myNavitem" href="https://github.com/nagyeszter/beadando" target="_blank"
            >GitHub</b-nav-item
          >
          <b-nav-item class="shadow-sm m-1 rounded myNavitem" href="https://github.com/nitslaszlo/JedlikComplexStarter" target="_blank"
            >SDK</b-nav-item
          >
        </b-navbar-nav>
      </b-collapse>
    </b-navbar>
    <b-container fluid>
      <b-row>
        <b-col lg="6">
          <TxtReader label="kerites.txt:" placeholder="Bedobó mező" class="my-3" @load="txtSorai = $event" />
          <div v-if="mutat" id="megoldas" class="shadow">
            <p>
              2. feladat<br />
              Az eladott telkek száma: {{ telkek.length }}
            </p>
            <p>
              3. feladat<br />
              A {{ utolsoTelek.oldal }} oldalon adták el az utolsó telket<br />
              Az utolsó telek házszáma: {{ utolsoTelek.hazSzama }}
            </p>
            <p>
              4. feladat<br />
              A szomszédossal egyezik a kerítés színe: {{ ugyanOlyanSzinuKerites }}
            </p>
            <p>
              5. feladat
              <b-row>
                <b-col sm="auto">
                  <label for="hazszamInput">Adjon meg egy házszámot!</label>
                </b-col>
                <b-col cols="3">
                  <b-form-input id="hazszamInput" v-model="hazszamInputStr" size="sm" type="number" min="1" max="117"></b-form-input>
                </b-col>
              </b-row>
              <br />
              A kerírés színe / állapota: {{ keritesSzineAllapota }}
              <br />
              Egy lehetséges festési szín: {{ lehetsegesFestesiSzin }}
            </p>
          </div>
        </b-col>
        <b-col lg="6">
          <div v-if="mutat">
            <TxtWriter title="utcakep.txt állomány mentése" :content="utcakep" filename="utcakep.txt" class="shadow" />
          </div>
          <div v-if="mutat" id="egyebek">
            utcakep.txt fájl:
            <div class="overflow-auto myBack1 shadow">
              <pre>{{ utcakep.trim() }}</pre>
            </div>
            kerites.txt fájl:
            <div class="overflow-auto myBack1 shadow" style="height:400px;">
              <pre>{{ txtSorai.trim() }}</pre>
            </div>
          </div>
        </b-col>
      </b-row>
    </b-container>
  </div>
</template>

<script lang="ts">
import { Component, Vue, Watch } from "vue-property-decorator";
import Telek from "./telek";
import TxtReader from "./components/TxtReader.vue";
import TxtWriter from "./components/TxtWriter.vue";

// eslint-disable-next-line
@Component({ components: { TxtReader, TxtWriter } })
export default class App extends Vue {
  private downloadPropObj = { download: "" };
  private telkek: Telek[] = [];
  private txtSorai: string = ""; // Watch végett nem lehet ékezetes azonosító! (pl.: forrás)!
  private mutat: boolean = false;
  // 5. feladat:
  private hazszamInputStr: string = "83";
  // 6. feladat:
  private utcakep: string = "";

  @Watch("txtSorai", { immediate: true, deep: true })
  private haForrasValtozik(val: string) {
    if (val !== "") this.feldolgoz();
  }

  private feldolgoz(): void {
    try {
      Telek.hazszamReset(); // statikus mezők visszaállítása
      this.txtSorai.split("\n").forEach(i => {
        const aktSor: string = i.trim();
        if (aktSor.length > 0) this.telkek.push(new Telek(aktSor));
      });
      // 6. utcakep generalasa
      let sor1: string = "";
      let sor2: string = "";
      for (const i of this.telkek.filter(x => x.paratlanOldali)) {
        sor1 += "".padEnd(i.telekSzelessege, i.keritesSzine);
        sor2 += i.hazSzama.toString().padEnd(i.telekSzelessege, " ");
      }
      this.utcakep = `${sor1}\r\n${sor2}\r\n`;

      this.mutat = true;
    } catch (error) {
      this.mutat = false;
      this.telkek = [];
      this.txtSorai = "";
      window.alert("Hibás forrás!");
      location.reload();
    }
  }

  private get utolsoTelek(): Telek {
    return this.telkek[this.telkek.length - 1];
  }

  private get ugyanOlyanSzinuKerites(): number {
    let elozoTelek: Telek; // induláskor undefined értékű
    // elozoKerites! -> igaz, ha értéke nem null vagy undefined
    for (const aktTelek of this.telkek.filter(x => x.paratlanOldali)) {
      if (elozoTelek! && aktTelek.keritesSzine !== "#" && aktTelek.keritesSzine !== ":" && aktTelek.keritesSzine === elozoTelek!.keritesSzine) {
        return elozoTelek!.hazSzama;
      } else elozoTelek = aktTelek; // ha még undefined az ElozoTelek
    }
    return -1; // id a feladatkiírás szerint már nem juthat el
  }

  private get keritesSzineAllapota(): string {
    const hazszamInput: number = parseInt(this.hazszamInputStr, 10);
    const keresettTelek: Telek[] = this.telkek.filter(x => x.hazSzama === hazszamInput);
    if (keresettTelek.length !== 0) {
      return keresettTelek[0].keritesSzine;
    } else {
      return "Nincs ilyen házszám!"; // ez nem volt feladat, de így szép
    }
  }

  private get lehetsegesFestesiSzin(): string {
    const hazszamInput: number = parseInt(this.hazszamInputStr, 10);
    const keresettTelek: Telek[] = this.telkek.filter(x => x.hazSzama === hazszamInput);
    const balSzomszedTelek: Telek[] = this.telkek.filter(x => x.hazSzama === hazszamInput + 2);
    const jobbSzomszedTelek: Telek[] = this.telkek.filter(x => x.hazSzama === hazszamInput - 2);
    let lehetsegesSzinek: string[] = ["A", "B", "C", "D"];
    if (keresettTelek.length !== 0) {
      // ha van telek, aminek a kerítését festeni kell
      lehetsegesSzinek = lehetsegesSzinek.filter(x => x !== keresettTelek[0].keritesSzine);
    } else return "Nincs ilyen házszám!";
    // ha van bal szomszéd:
    if (balSzomszedTelek.length !== 0) {
      lehetsegesSzinek = lehetsegesSzinek.filter(x => x !== balSzomszedTelek[0].keritesSzine);
    }
    // ha van jobb szomszéd:
    if (jobbSzomszedTelek.length !== 0) {
      lehetsegesSzinek = lehetsegesSzinek.filter(x => x !== jobbSzomszedTelek[0].keritesSzine);
    }
    return lehetsegesSzinek[0];
  }
}
</script>

<style>
#app {
  font-family: Consolas;
}

#myNavbar {
  background-color: burlywood;
}

.myNavitem {
  background-color: salmon;
  padding-left: 20px;
  padding-right: 20px;
}

.myNavitem:hover {
  background-color: beige;
}

.myBack1 {
  background-color: salmon;
  padding: 5px;
  border-radius: 10px;
}

#hazszamInput {
  background-color: burlywood;
  border: 1px solid black;
}

#megoldas {
  background-color: beige;
  padding: 10px;
  border-radius: 10px;
}

pre {
  font-size: 1em;
  margin: 0;
}
</style>
