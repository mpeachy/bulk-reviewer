<template>
  <section class="padded">
    <h1 class="title is-3">New session</h1>
    <form ref="form" @submit.prevent="onSubmit">
      
      <b-field label="Name">
        <b-input
          v-model="name"
          placeholder="Name, accession number, or other identifier"
          style="max-width: 700px;"
          required>
        </b-input>
      </b-field>

      <!-- Disk image? -->
      <b-field label="Source directory/disk image">
        <b-select
          placeholder="Select type"
          v-model="sourceType"
          required>
          <option value="directory">Directory</option>
          <option value="diskImage">Disk Image</option>
        </b-select>
      </b-field>

      <b-field>
        <section>
          <button
            class="button is-light"
            @click.prevent="chooseDirectory"
            v-if="sourceType === 'directory'">
            Choose directory
          </button>
          <button
            class="button is-light"
            @click.prevent="chooseFile"
            v-else>
            Choose file
          </button>
          <div style="margin: 5px;">
            <span v-if="sourcePath">
              {{ sourcePath }}
              <!-- button to reset sourcePath -->
              <button class="button is-light is-small" @click="resetSourcePath">
                <b-icon icon="times"></b-icon>
              </button>
            </span>
            <span v-else>
              None selected.
            </span>
          </div>
        </section>
    </b-field>

    <b-field label="Use existing bulk_extractor reports">
      <section>
        <div class="field">
          <b-checkbox v-model="useExistingBEReports">Use existing bulk_extractor reports directory</b-checkbox>
        </div>
        <button
          class="button is-light"
          @click.prevent="chooseBEReportsDir">
          Choose directory
        </button>
        <div style="margin: 5px;">
          <span v-if="bulkextractorReportsDirPath">
            {{ bulkextractorReportsDirPath }}
            <!-- button to reset sourcePath -->
            <button class="button is-light is-small" @click="resetBEReportsFilePath">
              <b-icon icon="times"></b-icon>
            </button>
          </span>
          <span v-else>
            None selected.
          </span>
        </div>
      </section>
    </b-field>

    <!-- Named entity extraction -->

    <b-field label="Regular expressions file">
      <section>
        <button
          class="button is-light"
          @click.prevent="chooseRegexFile">
          Choose file
        </button>
        <div style="margin: 5px;">
          <span v-if="regexFilePath">
            {{ regexFilePath }}
            <!-- button to reset sourcePath -->
            <button class="button is-light is-small" @click="resetRegexFilePath">
              <b-icon icon="times"></b-icon>
            </button>
          </span>
          <span v-else>
            None selected.
          </span>
        </div>
      </section>
    </b-field>

    <b-field label="Stoplist directory">
      <section>
        <button
          class="button is-light"
          @click.prevent="chooseStoplistDir">
          Choose directory
        </button>
        <div style="margin: 5px;">
          <span v-if="stoplistDirPath">
            {{ stoplistDirPath }}
            <!-- button to reset sourcePath -->
            <button class="button is-light is-small" @click="resetStoplistDirPath">
              <b-icon icon="times"></b-icon>
            </button>
          </span>
          <span v-else>
            None selected.
          </span>
        </div>
      </section>
    </b-field>

    <b-field label="Social Security Number identification mode">
      <b-select
        placeholder="SSN mode"
        v-model="ssnMode"
        required>
        <option value="0"><strong>Strict:</strong> must be labelled "SSN" (ssn_mode=0)</option>
        <option value="1"><strong>Medium:</strong> No “SSN” required; dashes required (ssn_mode=1)</option>
        <option value="2"><strong>Lenient:</strong> No dashes required (ssn_mode=2)</option>
      </b-select>
    </b-field>

    <b-field label="Scanner options">
      <section>
        <div class="field">
          <b-checkbox v-model="includeExifResults">Include EXIF metadata in results</b-checkbox>
        </div>
        <div class="field">
          <b-checkbox v-model="includeNetworkResults">Include network data (domains, URLs, RFC822 headers, HTTP logs) in results</b-checkbox>
        </div>
      </section>
    </b-field>

      <button 
        class="button is-success"
        @click.prevent="onSubmit"
        :disabled="isDisabled">
        Start scan
      </button>
      <button 
        class="button is-light"
        @click="resetForm">
        Reset
      </button>

    </form>

    <!-- Loading indicator -->
    <b-loading :is-full-page="true" :active.sync="loading" :can-cancel="false"></b-loading>

  </section>
</template>

<script>
const remote = require('electron').remote
const dialog = remote.dialog
const app = remote.app

export default {
  name: 'new-session',
  data () {
    return {
      isDisabled: false,
      name: '',
      sourceType: 'directory',
      sourcePath: '',
      bulkextractorReportsDirPath: '',
      regexFilePath: '',
      stoplistDirPath: '',
      ssnMode: '0',
      loading: false,
      includeExifResults: false,
      includeNetworkResults: false,
      useExistingBEReports: false
    }
  },
  methods: {
    chooseDirectory () {
      dialog.showOpenDialog({ properties: ['openDirectory'] }, (dirName) => {
        this.sourcePath = dirName.toString()
      })
    },
    chooseFile () {
      dialog.showOpenDialog({ properties: ['openFile'] }, (filename) => {
        this.sourcePath = filename.toString()
      })
    },
    chooseBEReportsDir () {
      dialog.showOpenDialog({ properties: ['openDirectory'] }, (dirName) => {
        this.bulkextractorReportsDirPath = dirName.toString()
      })
    },
    chooseRegexFile () {
      dialog.showOpenDialog({ properties: ['openFile'] }, (filename) => {
        this.regexFilePath = filename.toString()
      })
    },
    chooseStoplistDir () {
      dialog.showOpenDialog({ properties: ['openDirectory'] }, (dirName) => {
        this.stoplistDirPath = dirName.toString()
      })
    },
    resetSourcePath () {
      this.sourcePath = ''
    },
    resetBEReportsFilePath () {
      this.bulkextractorReportsDirPath = ''
    },
    resetRegexFilePath () {
      this.regexFilePath = ''
    },
    resetStoplistDirPath () {
      this.stoplistDirPath = ''
    },
    resetForm () {
      this.isDisabled = false
      this.name = ''
      this.sourceType = 'directory'
      this.sourcePath = ''
      this.bulkextractorReportsDirPath = ''
      this.regexFilePath = ''
      this.stoplistDirPath = ''
      this.ssnMode = '0'
      this.includeExifResults = false
      this.includeNetworkResults = false
      this.useExistingBEReports = false
    },
    // determine if python is packaged as executable by checking
    // for presence of backend_dist directory
    guessPyPackaged () {
      const path = require('path')
      const PY_DIST_FOLDER = path.join(__dirname, '../../main/', 'backend_dist')
      return require('fs').existsSync(PY_DIST_FOLDER)
    },
    // create and return Python subprocess
    // if python script has been packaged as executable, spawn it
    // otherwise, run python script using system python3
    createPythonProcess (sessionParameters) {
      if (app.isPackaged) {
        let pyExec = sessionParameters[0]
        let parametersMinusScript = sessionParameters.slice(1)
        let pyProc = require('child_process').spawn(pyExec, parametersMinusScript, { shell: true })
        return pyProc
      } else {
        let pyProc = require('child_process').spawn('python3', sessionParameters, { shell: true })
        return pyProc
      }
    },
    // get path to script or executable as appropriate
    getScriptPath () {
      const path = require('path')
      const PY_DIST_FOLDER = path.join(__dirname, '../../../backend_dist')
      const PY_FOLDER = path.join(__dirname, '../../main/', 'backend')
      const PY_MODULE = 'br_processor'
      // if app is built, get path to python exectuable in Resources
      if (app.isPackaged) {
        if (process.platform === 'win32') {
          return path.join(PY_DIST_FOLDER, PY_MODULE, PY_MODULE + '.exe')
        }
        return path.join(PY_DIST_FOLDER, PY_MODULE, PY_MODULE)
      // if not build, get path to script
      } else {
        return path.join(PY_FOLDER, PY_MODULE + '.py')
      }
    },
    // run br_processor python script. on completion, display errors
    // or, if no errors, load data from JSON file to store and
    // push router to ReviewDashboard component
    runPythonProcess () {
      const path = require('path')
      const homeDir = app.getPath('home')
      const brDir = path.join(homeDir, 'bulk-reviewer')

      let script = this.getScriptPath()
      let quotedScript = '"' + script + '"'

      let sessionParameters = [
        quotedScript,
        '--ssn',
        parseInt(this.ssnMode),
        '"' + this.sourcePath + '"',
        '"' + brDir + '"',
        '"' + this.name + '"'
      ]
      if (this.sourceType === 'diskImage') {
        sessionParameters.splice(1, 0, '-d')
      }
      if (this.includeExifResults === true) {
        sessionParameters.splice(1, 0, '--include_exif')
      }
      if (this.includeNetworkResults === true) {
        sessionParameters.splice(1, 0, '--include_network')
      }
      if (this.useExistingBEReports === true) {
        sessionParameters.splice(1, 0, '--be_reports')
        sessionParameters.splice(2, 0, this.bulkextractorReportsDirPath)
      }
      if (this.regexFilePath.length > 0) {
        sessionParameters.splice(1, 0, '--regex')
        sessionParameters.splice(2, 0, this.regexFilePath)
      }
      if (this.stoplistDirPath.length > 0) {
        sessionParameters.splice(1, 0, '--stoplists')
        sessionParameters.splice(2, 0, this.stoplistDirPath)
      }

      // fix for mac os to make system PATH available to node
      const fixPath = require('fix-path')
      fixPath()

      // create python process
      let pyProc = this.createPythonProcess(sessionParameters)

      // handle error starting python process
      if (!pyProc) {
        this.loading = false
        this.isDisabled = false
        this.errorMessage('Python process not started.')
      }

      // record stdout and stderr
      let jsonPath = ''
      let pyErrors = ''
      pyProc.stdout.on('data', function (data) {
        jsonPath += data.toString()
      })
      pyProc.stderr.on('data', function (data) {
        let errorMessage = data.toString()
        // ignore known non-error messages printed to stderr
        if (!errorMessage.includes('Attempt to open') &&
            !errorMessage.includes(' bytes') &&
            !errorMessage.includes('active scanners') &&
            !errorMessage.includes('lightgrep patterns')) {
          pyErrors += errorMessage
        }
      })

      // handle completion
      let self = this
      pyProc.stdout.on('end', function (data) {
        let jsonFile = jsonPath.trim()
        self.loading = false
        self.isDisabled = false

        // display errors
        if (pyErrors.length > 0) {
          self.errorMessage(`${pyErrors}`)
        }

        // if json path provided by backend, load review dashboard
        if (jsonFile.length > 0) {
          self.$store.dispatch('loadFromJSON', jsonFile)
          self.$router.push('review')
        }
      })
    },
    onSubmit () {
      // validate
      if (!this.name) {
        this.errorMessage('Name required')
        return
      }
      if (!this.sourcePath) {
        this.errorMessage('Source directory or disk image required')
        return
      }

      // disable submit button
      this.isDisabled = true

      // switch status indicators
      this.loading = true

      // kick off br_processor
      this.runPythonProcess()
    },
    // display error message in dialog
    errorMessage (msg) {
      this.$dialog.alert({
        title: 'Error',
        message: msg,
        type: 'is-danger',
        hasIcon: true,
        icon: 'times-circle',
        iconPack: 'fa'
      })
    }
  },
  computed: {
    brSession () {
      return this.$store.state.BRSession.brSession
    }
  }
}
</script>