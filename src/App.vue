<template>
  <div id="app">
    <loading v-if="!isModelLoaded"></loading>
    <div class="content columns">
      <div class="upload-wrapper column is-one-third">
        <div class="image-wrapper" v-show="img">
          <img id="source-image" ref="image" alt="Uploaded image" :src="img" />
          <div class="results">
            {{ result.label }}
            {{ result.confidence }}%
          </div>
        </div>
        <div class="file has-name is-boxed">
          <label class="file-label">
            <input
              class="file-input"
              type="file"
              name="uploadImage"
              @change="uploadImage"
            />
            <span class="file-cta">
              <span class="file-icon">
                <img
                  src="@/assets/_ionicons_svg_md-cloud-upload.svg"
                  alt="Upload icon"
                />
              </span>
              <span class="file-label">
                Choose a file...
              </span>
            </span>
            <span class="file-name">
              {{ imgName }}
            </span>
          </label>
        </div>
      </div>
      <div class="result-wrapper column" v-show="img">
        <div class="giphy-wrapper">
          <img :src="result.giphy" :alt="result.alt" v-if="result.giphy" />
          <a :href="result.giphyUrl" v-if="result.giphyUrl">via GIPHY</a>
        </div>

        <div class="style-wrapper">

          <div class="columns is-vcentered">
            <div class="column is-one-third">
              <a
                title="The Great Wave off Kanagawa, 1829 - Katsushika Hokusai"
                href="https://en.wikipedia.org/wiki/The_Great_Wave_off_Kanagawa"
                ><img
                  src="@/assets/wave.jpg"
                  style="width: 100px;"
                  alt="style one"
                /><br />Style A</a
              >
            </div>
            <div class="column">
              <span class="has-text-centered mini-loading" v-show="!isStyleAReady"
                ><img src="@/assets/timer.svg" class="loadicon" alt="Loading timer icon" />Applying
                Style A</span
              >

              <span id="styleA" v-show="isStyleAReady"></span>
            </div>
          </div>

          <div class="columns is-vcentered">
            <div class="column is-one-third">
              <a
                href="https://en.wikipedia.org/wiki/The_Great_Wave_off_Kanagawa"
                title="Udnie (Young American Girl, The Dance), 1913 - Francis Picabia"
                ><img
                  src="@/assets/udnie.jpg"
                  style="width: 100px;"
                  alt="style two"
                /><br />Style B</a
              >
            </div>
            
            <div class="column">
              <span class="has-text-centered mini-loading" v-show="!isStyleBReady"
                ><img src="@/assets/timer.svg" class="loadicon" alt="Loading timer icon" />Applying
                Style B</span
              >

              <span id="styleB" v-show="isStyleBReady"></span>
            </div>
          </div>
        </div>

      </div>
    </div>
    <footer class="footer">
      <div class="content has-text-centered">
        <p><a href="https://webidente.com">@webidente</a> 2020</p>
      </div>
    </footer>
  </div>
</template>

<script>
import ml5 from "ml5";
import axios from "axios";

import Loading from "./components/Loading.vue";

export default {
  name: "App",
  components: {
    Loading,
  },
  data() {
    return {
      classifier: {},
      isModelLoaded: false,
      isStyleAReady: false,
      isStyleBReady: false,
      result: {
        label: "",
        confidence: 0,
        giphy: "",
        giphyUrl: "",
        alt: "",
      },
      img: "",
      imgName: "Upload image",
    };
  },
  mounted: function () {
    this.classifier = ml5.imageClassifier("MobileNet", this.modelLoaded);
  },
  methods: {
    modelLoaded: function () {
      console.log("Model Loaded!");
      this.isModelLoaded = true;
    },
    classify: function () {
      // https://github.com/ml5js/ml5-library/blob/development/src/utils/IMAGENET_CLASSES.js
      this.classifier.classify(this.$refs.image, (err, results) => {
        if (err) {
          console.error(err);
        } else {
          console.log(results);
          this.result.label = results[0].label;
          this.result.confidence = (results[0].confidence * 100).toFixed(2);
          // this.getGiphy();
        }
      });
    },
    uploadImage: function (event) {
      const image = event.target.files[0];
      const reader = new FileReader();
      this.imgName = event.target.files[0].name;
      reader.readAsDataURL(image);
      reader.onload = (e) => {
        this.img = e.target.result;
        this.doStyleTransfer();
        this.classify();
      };
    },
    getGiphy: function () {
      // https://developers.giphy.com/docs/sdk/#web
      const url =
        "https://api.giphy.com/v1/gifs/search?api_key=gnHx8iIxLiE3MLUDnVWJ6pcWDlI8LGqL&limit=1&q=";
      this.result.giphy = require("@/assets/timer.svg");
      this.result.alt = "loading gif";
      this.result.giphyUrl = "";
      axios
        .get(
          url +
            this.result.label +
            "&offset=" +
            Math.floor(Math.random() * 100 + 1)
        )
        .then((response) => {
          const responseData = response.data.data[0];
          // console.log(responseData);
          this.result.giphy = responseData.images.original.url;
          this.result.alt = responseData.title;
          this.result.giphyUrl = responseData.url;
        })
        .catch((e) => {
          this.errors.push(e);
        });
    },

    doStyleTransfer: function () {
      const inputImg = document.getElementById("source-image"); // The image we want to transfer

      // const styleA = document.getElementById("styleA"); // container that holds new style image A
      // const styleB = document.getElementById("styleB"); // container that holds new style image B
      this.isStyleAReady = false;
      this.isStyleBReady = false;

      ml5
        .styleTransfer("models/wave")
        .then((style1) => style1.transfer(inputImg))
        .then((result) => {
          const newImage1 = new Image(320, 320);
          newImage1.src = result.src;
          const styleA = document.getElementById("styleA");
          styleA.innerHTML = "";
          styleA.appendChild(newImage1);
          console.log("style A:", styleA, newImage1);
          this.isStyleAReady = true;
        });

      ml5
        .styleTransfer("models/udnie")
        .then((style2) => style2.transfer(inputImg))
        .then((result) => {
          const newImage2 = new Image(320, 320);
          newImage2.src = result.src;
          const styleB = document.getElementById("styleB");
          styleB.innerHTML = "";
          styleB.appendChild(newImage2);
          console.log("style B:", styleB, newImage2);
          this.isStyleBReady = true;
          // statusMsg.innerHTML = "Done!";
        });
    },
  },
};
</script>

<style lang="scss">
@import "~bulma/css/bulma.css";

body {
  margin: 0;
}

#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;

  .content {
    margin: 0;
    height: calc(100vh - 80px);
  }

  .result-wrapper {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    margin-bottom: 80px;
    .giphy-wrapper {
      max-width: 90%;
      display: flex;
      flex-direction: column;
      align-items: center;
      img {
        max-width: 100%;
        max-height: 70vh;
      }
    }
    .style-wrapper {
      max-width: 90%;
      // display: flex;
      // flex-direction: column;
      align-items: center;
    }
  }

  .upload-wrapper {
    background-color: #efe;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    .image-wrapper {
      width: 70%;
      border-radius: 10px 10px 10px 10px;
      -moz-border-radius: 10px 10px 10px 10px;
      -webkit-border-radius: 10px 10px 10px 10px;
      border: 10px double #e8e8e8;
      padding: 5px;
      margin-bottom: 20px;
      img {
        max-width: 100%;
        max-height: 30vh;
      }
    }
    .file-name {
      text-align: center;
    }
  }
  .footer {
    position: fixed;
    bottom: 0;
    left: 0;
    width: 100%;
    height: 80px;
    background-color: #fff;
    padding: 1.5rem 1.5rem 3rem;
  }
}

#styleA,
#styleB {
  width: 360px;
  padding: 5px 0 0;
  display: block;

  img {
      border: 3px solid #333;
      background-color: #eee;
      padding: 8px;    
      border-radius: 10px 10px 10px 10px;
      -moz-border-radius: 10px 10px 10px 10px;
      -webkit-border-radius: 10px 10px 10px 10px;
  }
}

.mini-loading {
  font-size: 1.2em;
  display: flex;
  justify-content: center;
  align-items: center;
  img.loadicon {
    height: 32px;
    margin-right: 15px;
    animation-name: spin;
    animation-duration: 3000ms;
    animation-iteration-count: infinite;
    animation-timing-function: linear;
  }
}

@-moz-keyframes spin {
  from {
    -moz-transform: rotate(0deg);
  }
  to {
    -moz-transform: rotate(360deg);
  }
}
@-webkit-keyframes spin {
  from {
    -webkit-transform: rotate(0deg);
  }
  to {
    -webkit-transform: rotate(360deg);
  }
}
@keyframes spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}
</style>
