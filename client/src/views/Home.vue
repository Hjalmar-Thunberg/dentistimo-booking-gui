<template>
  <div id="home">
    <b-col>
      <p id="dentisimo" class="headlines">Dentistimo</p>
      <p id="available" class="headlines">
        Your <span id="available"> available </span> local dentist office
      </p>
      <b-button id="book" v-bind:href="'/booking/'">BOOK A TIME</b-button>
      <p id="or">or</p>
      <p id="find">FIND US HERE</p>

      <b-row align-h="left">
        <div id="map-div">
          <Map id="map" ref="Map" :offices="offices" />
        </div>
      </b-row>
    </b-col>
  </div>
</template>

<script>
import Map from "../components/map.vue";

export default {
  name: "Home",
  components: {
    Map,
  },
  mounted() {
    this.$mqtt.on("close", (err) => {
      if (err) console.log(err);
      console.log("close");
      this.$mqtt.unsubscribe("dentistimo/dentists");
    });
    this.$mqtt.on("offline", (err) => {
      if (err) console.log(err);
      console.log("offline");
      this.$mqtt.unsubscribe("dentistimo/dentists");
    });

    this.$mqtt.on("connect", (connack) => {
      if (connack.sessionPresent == false) {
        this.$mqtt.publish(
          "dentistimo/dentistoffice",
          JSON.stringify({ method: "getAll" }),
          1
        );
        this.$mqtt.subscribe("dentistimo/dentists");
      }
    });
  },
  mqtt: {
    // Retrieves the data of all dentist offices.
    "dentistimo/dentists"(data) {
      let jsonData = JSON.parse(data);
      if (jsonData != null) {
        this.offices = jsonData;
      } else {
        console.log("empty jsonString recieved");
        let message =
          "empty mqtt jsonString sent to bookingGUI via the broker. on topic: dentistimo/dentists ";
        this.$mqtt.publish("dentistimo/log/error", message, 2);
      }
    },
  },
  data() {
    return {
      offices: [],
    };
  },
};
</script>

<style>
@import url("https://fonts.googleapis.com/css2?family=Libre+Baskerville:ital@1&display=swap");

@media screen and (max-width: 600px){
  #or {
    display: none;
  }
  #find {
    display: none;
  }
  #book {
    margin-top: 1em;
  }
}

#available {
  font-style: italic;
}

#decription {
  margin: 2em;
}

.headlines {
  font-family: "Libre Baskerville", serif;
}

#dentisimo {
  font-size: 60px;
}

#available {
  font-size: 25px;
}

#book {
  border-radius: 30px;
  margin-top: 1em;
  padding: 18px 50px;
  background-color: #e98d58;
  border: #e98d58;
  font-size: 20px;
  font-family: "Libre Baskerville", serif;
}

#book:hover {
  color: rgb(57, 77, 41);
}

p {
  font-family: "Libre Baskerville", serif;
}

#or {
  margin-top: 0.5em;
  font-size: 20px;
}

#find {
  font-size: 22px;
}

.b-container fluid {
  padding-right: 0px;
  padding-left: 0px;
  margin-right: auto;
  margin-left: auto;
}

#home {
  font-family: serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

</style>
