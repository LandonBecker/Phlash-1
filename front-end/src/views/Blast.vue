<template>
  <div class="wrapper">
    <Navbar :upload="navUpload" :dnamaster="navDNAMaster" :blast="navBlast" :annotations="navAnnotations" />
    <div class="container">
      <h1>BLAST</h1>
      <div class="alert alert-primary">
        <p><strong>Instructions</strong></p>
        <p>Follow the steps below. Continue when all steps have been completed.</p>
        <div class="nav-btns-wrapper">
          <router-link :to="{ name: 'DNAMaster', params: {phageID: $route.params.phageID} }">
            <button class="btn btn-light btn-nav">
              <svg class="bi bi-arrow-left" width="1em" height="1em" viewBox="0 0 16 16" fill="currentColor" xmlns="http://www.w3.org/2000/svg">
                <path fill-rule="evenodd" d="M5.854 4.646a.5.5 0 010 .708L3.207 8l2.647 2.646a.5.5 0 01-.708.708l-3-3a.5.5 0 010-.708l3-3a.5.5 0 01.708 0z" clip-rule="evenodd"/>
                <path fill-rule="evenodd" d="M2.5 8a.5.5 0 01.5-.5h10.5a.5.5 0 010 1H3a.5.5 0 01-.5-.5z" clip-rule="evenodd"/>
              </svg>
              <strong>Back</strong>
            </button>
          </router-link>
          <router-link :to="{ name: 'Annotations', params: {phageID: $route.params.phageID} }"
                       :event="blast ? 'click' : ''">
            <button class="btn btn-light btn-nav disabled" id="next-top">
              <strong>Next</strong>
              <svg class="bi bi-arrow-right" width="1em" height="1em" viewBox="0 0 16 16" fill="currentColor" xmlns="http://www.w3.org/2000/svg">
                <path fill-rule="evenodd" d="M10.146 4.646a.5.5 0 01.708 0l3 3a.5.5 0 010 .708l-3 3a.5.5 0 01-.708-.708L12.793 8l-2.647-2.646a.5.5 0 010-.708z" clip-rule="evenodd"/>
                <path fill-rule="evenodd" d="M2 8a.5.5 0 01.5-.5H13a.5.5 0 010 1H2.5A.5.5 0 012 8z" clip-rule="evenodd"/>
              </svg>
            </button>
          </router-link>
        </div>
      </div>
      <div class="steps">
        <ol>
          <li class="step">
            Download the FASTA file that will be used as input for BLAST.<br />
              <button class="btn btn-light" style="position: relative;" @click="downloadFile">
                <loading :active.sync="downloadLoading" :is-full-page="false" :height="20" :width="20"></loading>
                <strong>Download FASTA file</strong>
              </button>
          </li>
          <li class="step">
            Go to BLASTp's website. <i>At the website, make sure to upload
              your file and set the appropriate parameters, as show in the list
              and screenshot below.</i><br />
            <ul>
              <li><strong>Upload File:</strong> Upload the FASTA file from step 1.</li>
              <li><strong>Database:</strong> Non-redundant protein sequences (nr)</li>
              <li><strong>Algorithm:</strong> blastp (protein-protein BLAST)</li>
            </ul>
            <div class="alert alert-warning" v-if="clickedNCBI">
              This will take several minutes. If it seems to be taking a long time, 
              it is still probably working correctly. For help troubleshooting, visit the FAQ.
              <button class="btn btn-light btn-step" @click="goToHelp">
                <strong>Go to BLAST help page</strong>
              </button>
            </div>
            <button class="btn btn-light btn-step" @click="goToNCBI">
              <strong>Go to NCBI's BLASTp</strong>
            </button>
            <!-- <img id="step-two" src="<%= BASE_URL %>/images/blast_step2.png" /> -->
            <img id="step-two" src="/phlash/images/blast_step2.png" />
          </li>
          <li class="step">
            In the top left table on the results page, click on
            <i>"Download All."</i> This will show you file formatting options for
            downloading your results. Choose <strong>Single-file JSON</strong>. 
            Continue when you have your downloaded file ready for upload.<br />
            <!--<img id="step-three" src="<%= BASE_URL %>/images/blast_step3.png" />-->
            <img id="step-three" src="/phlash/images/blast_step3.png" />
          </li>
          <li class="step">
            Upload your BLAST results.
            <div class="upload-wrapper">
              <div class="alert alert-warning" id="blast-warning-alert" role="alert" v-if="showNoMoreFiles"></div>
              <div class="alert alert-success" id="blast-success-alert" role="alert" v-if="showBlastSuccessAlert"></div>
              <div class="alert alert-danger" id="blast-danger-alert" role="alert" v-if="showBlastDangerAlert"></div>
              <div class="upload">
                <form id="blast-upload-form" role="form" enctype="multipart/form-data">
                  <div class="upload-btn-wrapper">
                    <button class="btn btn-upload btn-step">
                      <loading :active.sync="blastLoading" :is-full-page="false" :height="80" :width="80"></loading>
                      Drag files here or click to browse <br />
                      <div class="selected-file" v-if="showBlastFile">
                        <strong>Selected file: {{ this.blastFile.name }}</strong>
                      </div>
                    </button>
                    <input class="form-control" id="file" type="file" ref="file" name="file" v-on:change="handleFileUpload()" />
                  </div>
                </form>
              </div>
              <button class="btn btn-dark btn-upload-submit" v-if="showBlastFile" @click="uploadFile()">
                <strong>Upload</strong>
              </button>
              <div class="file-list">
                <ul>
                  <li v-for="jsonFile in jsonFiles" v-bind:key="jsonFile">
                    <button class="btn btn-light btn-download" @click="downloadJsonFile(jsonFile)">
                      <strong>Download</strong> {{ jsonFile.name }}
                      <svg width="2em" height="2em" viewBox="0 0 16 16" class="bi bi-download" fill="currentColor" xmlns="http://www.w3.org/2000/svg">
                        <path fill-rule="evenodd" d="M.5 9.9a.5.5 0 0 1 .5.5v2.5a1 1 0 0 0 1 1h12a1 1 0 0 0 1-1v-2.5a.5.5 0 0 1 1 0v2.5a2 2 0 0 1-2 2H2a2 2 0 0 1-2-2v-2.5a.5.5 0 0 1 .5-.5z"/>
                        <path fill-rule="evenodd" d="M7.646 11.854a.5.5 0 0 0 .708 0l3-3a.5.5 0 0 0-.708-.708L8.5 10.293V1.5a.5.5 0 0 0-1 0v8.793L5.354 8.146a.5.5 0 1 0-.708.708l3 3z"/>
                      </svg>
                    </button>
                    <button class="btn btn-light btn-trash" @click="deleteFiles(jsonFile)">
                      <strong>Delete</strong> {{ jsonFile.name }}
                      <svg width="2em" height="2em" viewBox="0 0 16 16" class="bi bi-trash" fill="this.backgroundColor" xmlns="http://www.w3.org/2000/svg">
                        <path d="M5.5 5.5A.5.5 0 0 1 6 6v6a.5.5 0 0 1-1 0V6a.5.5 0 0 1 .5-.5zm2.5 0a.5.5 0 0 1 .5.5v6a.5.5 0 0 1-1 0V6a.5.5 0 0 1 .5-.5zm3 .5a.5.5 0 0 0-1 0v6a.5.5 0 0 0 1 0V6z"/>
                        <path fill-rule="evenodd" d="M14.5 3a1 1 0 0 1-1 1H13v9a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V4h-.5a1 1 0 0 1-1-1V2a1 1 0 0 1 1-1H6a1 1 0 0 1 1-1h2a1 1 0 0 1 1 1h3.5a1 1 0 0 1 1 1v1zM4.118 4L4 4.059V13a1 1 0 0 0 1 1h6a1 1 0 0 0 1-1V4.059L11.882 4H4.118zM2.5 3V2h11v1h-11z"/>
                      </svg>
                    </button>
                  </li>
                </ul>
              </div>
            </div>
          </li>
        </ol>
      </div>
      <div class="nav-btns-wrapper">
        <router-link :to="{ name: 'DNAMaster', params: {phageID: $route.params.phageID} }">
          <button class="btn btn-light btn-nav">
            <svg class="bi bi-arrow-left" width="1em" height="1em" viewBox="0 0 16 16" fill="currentColor" xmlns="http://www.w3.org/2000/svg">
              <path fill-rule="evenodd" d="M5.854 4.646a.5.5 0 010 .708L3.207 8l2.647 2.646a.5.5 0 01-.708.708l-3-3a.5.5 0 010-.708l3-3a.5.5 0 01.708 0z" clip-rule="evenodd"/>
              <path fill-rule="evenodd" d="M2.5 8a.5.5 0 01.5-.5h10.5a.5.5 0 010 1H3a.5.5 0 01-.5-.5z" clip-rule="evenodd"/>
            </svg>
            <strong>Back</strong>
          </button>
        </router-link>
        <router-link :to="{ name: 'Annotations', params: {phageID: $route.params.phageID} }"
                      :event="blast ? 'click' : ''">
          <button class="btn btn-light btn-nav disabled" id="next-bottom">
            <strong>Next</strong>
            <svg class="bi bi-arrow-right" width="1em" height="1em" viewBox="0 0 16 16" fill="currentColor" xmlns="http://www.w3.org/2000/svg">
              <path fill-rule="evenodd" d="M10.146 4.646a.5.5 0 01.708 0l3 3a.5.5 0 010 .708l-3 3a.5.5 0 01-.708-.708L12.793 8l-2.647-2.646a.5.5 0 010-.708z" clip-rule="evenodd"/>
              <path fill-rule="evenodd" d="M2 8a.5.5 0 01.5-.5H13a.5.5 0 010 1H2.5A.5.5 0 012 8z" clip-rule="evenodd"/>
            </svg>
          </button>
        </router-link>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import Vue from "vue";
import Navbar from "../components/Navbar.vue";
import Loading from "vue-loading-overlay";
import "vue-loading-overlay/dist/vue-loading.css";

export default {
  name: "Blast",
  components: {
    Loading,
    Navbar
  },
  data() {
    return {
      downloadLoading: false,
      blastLoading: false,
      fileDownloaded: false,
      clickedNCBI: false,
      blast: false,
      blastFile: null,
      jsonFiles: [],
      showBlastFile: false,
      showBlastDangerAlert: false,
      showBlastSuccessAlert: false,
      showDeleteIcon: false,
      numFiles: 1,
      showNoMoreFiles: false,
    };
  },

  created() {
    this.checkIfFileUploaded();
    this.displayFiles();
  },

  computed: {
    navUpload: function() {
      return true;
    },
    navDNAMaster: function() {
      return true;
    },
    navBlast: function() {
      return true;
    },
    navAnnotations: function() {
      if (this.blast) return true;
      else return false;
    },
  },

  watch: {
    blast: function() {
      if (this.blast) {
        document.getElementById("next-top").classList.remove("disabled");
        document.getElementById("next-bottom").classList.remove("disabled");
      }
    },
  },

  methods: {
    checkIfFileUploaded() {
      axios.get(process.env.VUE_APP_BASE_URL + `/blast/${this.$route.params.phageID}/none`)
        .then(response => {
          this.blast = response.data.blast;
        })
        .catch(error => {
          console.log(error);
        })
    },

    downloadJsonFile(file) {
      var data = new FormData();
        data.append("file", file);
        axios.post(process.env.VUE_APP_BASE_URL + `/blast/${this.$route.params.phageID}/downloadJSON`,
            data,
            {
              headers: {
                "Content-Type": "multipart/form-data"
              }
            }
          )
          .then(response => {
          let data = response.data;
          const blob = new Blob([data], { type: "application/fasta" });
          let link = document.createElement("a");
          link.href = window.URL.createObjectURL(blob);
          link.download = `${file.name}`;
          link.click();
          this.downloadLoading = false;
          this.fileDownloaded = true;
        });
    },

    downloadFile() {
      this.downloadLoading = true;
      axios.post(process.env.VUE_APP_BASE_URL + `/blast/${this.$route.params.phageID}/download`)
        .then(response => {
          let data = response.data;
          const blob = new Blob([data], { type: "application/fasta" });
          let link = document.createElement("a");
          link.href = window.URL.createObjectURL(blob);
          link.download = `${this.$route.params.phageID}_blast.fasta`;
          link.click();
          this.downloadLoading = false;
          this.fileDownloaded = true;
        });
    },

    getName() {
      axios.post(process.env.VUE_APP_BASE_URL + `/blast/${this.$route.params.phageID}/name`)
        .then(response => {
            let data = response.data;
            console.log(data);
            return response.data;
          });
    },

    displayFiles() {
      axios.post(process.env.VUE_APP_BASE_URL + `/blast/${this.$route.params.phageID}/display`)
        .then(response => {
          let data = response.data;
          if (data != "No file uploaded") {
            const blob = new Blob([data], { type: "application/blast" });
            axios.post(process.env.VUE_APP_BASE_URL + `/blast/${this.$route.params.phageID}/name`)
              .then(response => {
                let name = response.data;
                console.log(name);
                var file = new File([blob], name);
                this.jsonFiles.push(file);
              })
          }
        })
        .catch(error => {
          console.log(error);
        })
    },

    deleteFiles(jsonFile) {
      this.showNoMoreFiles = false;
      var data = new FormData();
      data.append("file", jsonFile);
      axios.post(process.env.VUE_APP_BASE_URL + `/blast/${this.$route.params.phageID}/remove`,
          data,
          {
            headers: {
              "Content-Type": "multipart/form-data"
            }
          }
        )
        .then(response => {
          var index = this.jsonFiles.indexOf(jsonFile);
          this.jsonFiles.splice(index, 1);
          console.log(response);
          if (typeof response.data.uploaded !== "undefined") {
          } else if (typeof response.data.not_allowed !== "undefined") {
            let fileExt = response.data.not_allowed.split(".").pop();
          }
        })
        .catch(error => {
          console.log(error);
        });
    },

    goToNCBI() {
      window.open(
        "https://blast.ncbi.nlm.nih.gov/Blast.cgi?PAGE=Proteins",
        "_blank"
      );
      this.clickedNCBI = !this.clickedNCBI;
    },

    goToHelp() {
      window.open(
        "https://blast.ncbi.nlm.nih.gov/Blast.cgi?CMD=Web&PAGE_TYPE=Blastdocs",
        "_blank"
      );
    },

    handleFileUpload() {
      this.blastFile = document.querySelector(
        "#blast-upload-form"
      ).file.files[0];
      this.showBlastFile = true;
    },

    uploadFile(e) {
      if (this.jsonFiles.length == this.numFiles) {
        this.showBlastDangerAlert = false;
        this.showBlastSuccessAlert = false;
        this.showNoMoreFiles = true;
        let warningMessage = "You must remove a file before uploading another.";
        Vue.nextTick(() => {
                document.getElementById(
                  "blast-warning-alert"
                ).innerHTML = warningMessage;
        });
      }
      else {
        this.showNoMoreFiles = false;
        this.blastLoading = true;
        var data = new FormData();
        data.append("file", this.blastFile);
        axios.post(process.env.VUE_APP_BASE_URL + `/blast/${this.$route.params.phageID}/upload`,
            data,
            {
              headers: {
                "Content-Type": "multipart/form-data"
              }
            }
          )
          .then(response => {
            console.log(response);
            if (typeof response.data.uploaded !== "undefined") {
              this.jsonFiles.push(this.blastFile);
              this.blastLoading = false;
              this.showBlastSuccessAlert = true;
              this.showBlastDangerAlert = false;
              this.blast = true;
              let successMessage = `<strong>${response.data.uploaded}</strong> uploaded successfully!`;
              Vue.nextTick(() => {
                document.getElementById(
                  "blast-success-alert"
                ).innerHTML = successMessage;
              });
            } else if (typeof response.data.not_allowed !== "undefined") {
              let fileExt = response.data.not_allowed.split(".").pop();
              this.blastLoading = false;
              this.showBlastDangerAlert = true;
              this.showBlastSuccessAlert = false;
              let dangerMessage = `<strong>${fileExt}</strong> is an unacceptable JSON file extension.`;
              Vue.nextTick(() => {
                document.getElementById(
                  "blast-danger-alert"
                ).innerHTML = dangerMessage;
              });
            }
          })
          .catch(error => {
            console.log(error);
          });
      }
    }
  }
};
</script>

<style scoped>
.wrapper {
  margin: 0;
}

/* ----- Title and Alerts ----- */
h1 {
  margin: 40px auto;
}

.alert-primary {
  text-align: left;
  margin: 40px auto;
}

/* ----- Steps ----- */
.steps {
  text-align: left;
}

.step {
  margin-bottom: 20px;
}

.steps img {
  max-width: 100%;
  height: auto;
  margin: 10px;
}

.btn-step {
  margin: 15px auto;
}

.nav-btns-wrapper {
  text-align: center;
}

.btn-nav {
  margin: 5px;
}

.bi-arrow-left {
  margin-right: 5px;
  margin-left: 0px;
}

.bi-arrow-right {
  margin-right: 0px;
  margin-left: 5px;
}

/* ----- Upload ----- */
.upload-wrapper {
  margin: 30px auto;
}

.upload-btn-wrapper {
  position: relative;
  overflow: hidden;
  display: inline-block;
  width: 100%;
}

.btn-upload {
  border: 3px dashed gray;
  color: gray;
  background-color: white;
  padding-top: 20px;
  padding-bottom: 20px;
  width: 100%;
  border-radius: 8px;
  font-size: 18px;
  font-weight: bold;
}

.upload-btn-wrapper input[type="file"] {
  font-size: 100px;
  position: absolute;
  left: 0;
  top: 0;
  opacity: 0;
}

.selected-file {
  display: inline-block;
  margin: 10px;
}

.selected-file strong {
  color: #474747;
  font-size: 16px;
  text-align: center;
}

.selected-file span {
  margin: 0;
  color: #474747;
  font-size: 16px;
  text-align: left;
}

.btn-upload-submit {
  display: block;
  margin: auto;
  width: 100%;
}

.btn-download:hover {
  color: rgb(26, 87, 26);
}

.btn-trash:hover {
  color: rgb(143, 27, 27);
}

.btn-download {
  width: 48%;
  margin: 5px;
}

.btn-trash {
  width: 48%;
  margin: 5px;
}

.file-list {
  padding-top: 20px;
}

.alert-warning {
  margin-top: 20px;
}

</style>
