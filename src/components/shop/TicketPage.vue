<template>
  <v-container>
    <v-row class="text-center" justify="center">
      <v-col cols="12">
        <p>BILETY:</p>
        <!--        SEANSA ID: {{ cart[0].seans }}<br />-->
        <!--        KOSZYK: {{ cart }}<br />-->
        <!--        WYBRANE FILMY: {{ films_array }}<br />-->
        <!--        UZYTKOWNIK: {{ USSER }}<br />-->
<!--        CENY: {{ prizes_array }}-->
      </v-col>
      <v-col>
        <v-card v-for="(place, index) in cart[0].miejsca" :key="index">
          <template v-if="place">
            <v-col cols v-for="(one, index1) in place.miejsca" :key="index1">
              Nazwa Filmu: {{ films_array[0].tytul }} <br />
              Rząd: {{ one.lay }} <br />
              Miejsce: {{ one.number }} <br />
              <v-col cols="4" align-self="center" offset="4">
                Typ Biletu:
                <v-select
                  v-model="select_value[index1]"
                  :items="biletType"
                  return-object
                  item-text="nazwa"
                >
                </v-select>
              </v-col>
            </v-col>
          </template>
        </v-card>
      </v-col>
    </v-row>
    <v-row class="text-center" justify="center">
            <template v-if="userLogged">
      <v-btn color="primary" dark @click.stop="dialog = true">
        Dodatki
      </v-btn>
      <v-dialog v-model="dialog" max-width="500">
        <v-card>
          <v-card-title class="headline"
            >Jakie dodatki chcesz wybrać?</v-card-title
          >

          <v-card-text>
            <v-col v-for="(dodatek, index) in Dodatki" :key="index">
              <b>{{ dodatek.nazwa }}</b> <br />
              Cena: <b>{{ dodatek.cena }} zł</b><br />
              <v-text-field
                v-model="dodatki_numbers[index]"
                type="number"
                label="Ilość:"
              />
            </v-col>
          </v-card-text>

          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="green darken-1" text @click="dialog = false">
              Kup
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
      <v-col cols="12">
        <v-btn @click="make_object_array(cart[0].miejsca)">ZAPALAC</v-btn>
      </v-col>
            </template>
            <template v-else>
              <h1>NAJPIERW ZALOGUJ SIE ABY ZAPŁACIĆ</h1>
            </template>
    </v-row>
  </v-container>
</template>

<script>
import axios from "axios";
// import { apiService } from "@/common/api.service";
import { CSRF_TOKEN } from "@/common/csrf_token";
// import {apiService} from "@/common/api.service";

export default {
  name: "TicketPage",

  data: () => ({
    reserved_object_array: [],
    films_array: [],
    prizes_array: [],
    select_value: [],
    numberValue: 0,
    dodatki_numbers: [],
    dialog: false
  }),
  mounted() {
    this.getFilm(this.cart[0].seans.film);
    this.getPrizes();
  },
  computed: {
    userLogged() {
      return this.$store.state.ifLogin;
    },
    USSER() {
      return this.$store.state.userOnline;
    },
    cart() {
      return this.$store.state.cart;
    },
    biletType() {
      return this.prizes_array
        .filter(x => {
          if (x.cena_id === 1 || x.cena_id === 2) return x;
        })
        .map(x => x);
    },
    Dodatki() {
      return this.prizes_array
        .filter(x => {
          if (x.cena_id === 3 || x.cena_id === 4) return x;
        })
        .map(x => x);
    }
  },
  methods: {
    getPrizes() {
      axios
        .get(`api/prizes/`, {
          headers: { "X-CSRFTOKEN": CSRF_TOKEN }
        })
        .then(response => {
          this.prizes_array = response.data;
        });
    },
    getFilm(id) {
      axios
        .get(`api/films/${id}/`, {
          headers: { "X-CSRFTOKEN": CSRF_TOKEN }
        })
        .then(response => {
          this.films_array.push(response.data);
        });
    },
    make_object_array(arrray) {
      for (let i = 0; i < arrray.length; i += 1) {
        if (arrray[i] != null) {
          for (let j = 0; j < arrray[i].miejsca.length; j += 1) {
              console.log(j)
            const Data = {
              client: this.USSER.id,
              row: arrray[i].miejsca[j].lay,
              column: arrray[i].miejsca[j].number,
              code: arrray[i].miejsca[j].code,
              filmName: this.cart[0].seans.film,
              seans_id: this.cart[0].seans.seans_id,
              ticktet_type: this.select_value[j].nazwa,
              prize: this.prizes_array[0].cena,
              prize_id: this.prizes_array[0].cena_id,
              ticket_name: `${arrray[i].miejsca[j].code}-${this.select_value[j].nazwa}`
            };
            console.log(Data);
              console.log(this.select_value);
            // this.reservedPlace(Data);
            // this.sendTicket(Data);
            this.$router.push("/information2");
          }
        }
      }
    },

    async reservedPlace(DATA) {
      console.log(DATA.code);
      console.log(DATA.seans_id);
      axios
        .post(
          "api/reserved/",
          {
            adres_miejsca: DATA.code,
            seans: DATA.seans_id
          },
          {
            headers: { "X-CSRFTOKEN": CSRF_TOKEN }
          }
        )
        .then(function(response) {
          console.log(response);
        })
        .catch(function(error) {
          console.log(error);
        });
    },

    async sendTicket(Data) {
      console.log(Data.ticket_name);
      console.log(Data.seans_id);
      console.log(Data.prize_id);
      // let endpoint = "api/ticket/";
      // let method = "POST";
      // const testData = {
      //   client: "user",
      //   row: "A",
      //   column: 10,
      //   seans_name: 1
      // };
      // apiService(method, endpoint, {
      //   client: "user",
      //   row: "A",
      //   column: 10,
      //   seans_name: 1
      // })
      //   .then(function(response) {
      //     console.log(response);
      //   })
      //   .catch(function(error) {
      //     console.log(error);
      //   });
      axios
        .post(
          "/api/ticket/",

          {
            rodzaj_biletu: Data.ticket_name,
            seans: Data.seans_id,
            cena: Data.prize_id
          },
          {
            headers: { "X-CSRFTOKEN": CSRF_TOKEN }
          }
        )
        .then(function(response) {
          console.log(response);
        })
        .catch(function(error) {
          console.log(error);
        });
    }
  }
};
</script>

<style scoped></style>
