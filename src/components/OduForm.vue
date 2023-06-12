<template>
  <div class="container">
    <div class="function-form">
      <div class="form-group">
        <label for="functionInput">Function</label>
        <select id="functionInput" class="form-control" v-model="equation_id">
          <option value="1" selected>x+y</option>
          <option value="2">2*x*y</option>
          <option value="3">e^x - 2*y</option>
        </select>
      </div>
      <div class="form-group">
        <label for="x0">X_0</label>
        <input ref="x0" id="x0" type="number" class="form-control" v-model="x0"/>
        <div v-if="errorMessages['x0']" class="error-message">{{ errorMessages['x0'] }}</div>
      </div>
      <div class="form-group">
        <label for="xn">X_N</label>
        <input ref="xn" id="xn" type="number" class="form-control" v-model="xn"/>
        <div v-if="errorMessages['xn']" class="error-message">{{ errorMessages['xn'] }}</div>
      </div>
      <div class="form-group">
        <label for="y0">Y_0</label>
        <input ref="y0" id="y0" type="number" class="form-control" v-model="y0"/>
        <div v-if="errorMessages['y0']" class="error-message">{{ errorMessages['y0'] }}</div>
      </div>
      <div class="form-group">
        <label for="step">Step</label>
        <input ref="step" type="number" id="step" class="form-control" v-model="step">
        <div v-if="errorMessages['step']" class="error-message">{{ errorMessages['step'] }}</div>
      </div>
      <div class="form-group">
        <label for="accuracy">Accuracy</label>
        <input ref="accuracy" type="number" id="step" class="form-control" v-model="accuracy">
        <div v-if="errorMessages['accuracy']" class="error-message">{{ errorMessages['accuracy'] }}</div>
      </div>
    </div>
    <div class="buttons-container">
      <button :title="errorMessage" @click="sendOdu" class="send-btn">
        Solve
      </button>
    </div>
    <div v-if="errorMessage" class="error-message">{{ errorMessage }}</div>

    <div ref="plot"/>

    <h1>EXACT</h1>
    <h2>i = {{ exact.length }}</h2>
  <table>
    <thead>
      <tr>
        <th>X</th>
        <th>Y</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="point in exact" :key="point.x">
        <td>{{ point.x.toFixed(3) }}</td>
        <td>{{ point.y.toFixed(3) }}</td>
      </tr>
    </tbody>
  </table>
  <br><br>
  <h1>EULER</h1>
  <h2>i = {{ euler.length }}</h2>
  <table>
    <thead>
      <tr>
        <th>X</th>
        <th>Y</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="point in euler" :key="point.x">
        <td>{{ point.x.toFixed(3) }}</td>
        <td>{{ point.y.toFixed(3) }}</td>
      </tr>
    </tbody>
  </table>
  <br><br>
  <h1>MILN</h1>
  <h2>i = {{ miln.length }}</h2>
  <table>
    <thead>
      <tr>
        <th>X</th>
        <th>Y</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="point in miln" :key="point.x">
        <td>{{ point.x.toFixed(3) }}</td>
        <td>{{ point.y.toFixed(3) }}</td>
      </tr>
    </tbody>
  </table>
  <br><br>
  <h1>RUNGE</h1>
  <h2>i = {{ runge.length }}</h2>
  <table>
    <thead>
      <tr>
        <th>X</th>
        <th>Y</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="point in runge" :key="point.x">
        <td>{{ point.x.toFixed(3) }}</td>
        <td>{{ point.y.toFixed(3) }}</td>
      </tr>
    </tbody>
  </table>

  </div>
</template>

<script>
import axios from 'axios';
import Plotly from 'plotly.js-dist';

export default {
  data() {
    return {
      func: 'Math.exp(x)',
      equation_id: '1',
      x0: '',
      y0: '',
      errorMessages: {},
      errorMessage: '',
      functions: [],
      xn: '',
      step: '',
      accuracy: '',
      response: '',
      euler: [],
      exact: [],
      miln: [],
      runge: []
    };
  },
  watch: {
    func: 'renderPlot'
  },
  methods: {
    isValid() {
      let s = "All fields must not be empty";
      if (this.x0 === '') {
        this.errorMessage = s;
      } else {
        if (this.x0 >= this.xn) {
          this.errorMessages['x0'] = "x0 must be less then xn"
        }
      }

      if (this.y0 === '') {
        this.errorMessage = s;
      }

      if (this.xn === '') {
        this.errorMessage = s;
      } else {
        if (this.x0 >= this.xn) {
          this.errorMessages['xn'] = "x0 must be less then xn"
        }
      }

      if (this.step === '') {
        this.errorMessage = s;
      } else {
        if (this.step <= 0) {
          this.errorMessages['step'] = "Step must be positive";
          this.errorMessage = "Fix errors above, please";
        }
        if (this.step >= 20) {
          this.errorMessages['step'] = "Step must be less then 20";
          this.errorMessage = "Fix errors above, please";
        }
      }

      if (this.accuracy === '') {
        this.errorMessage = s;
      } else {
        if (this.accuracy <= 0) {
          this.errorMessages['accuracy'] = "Accuracy must be positive";
          this.errorMessage = "Fix errors above, please";
        }
      }

      return this.errorMessage === '';
    },
    sendOdu() {
      this.errorMessages = {};
      this.errorMessage = '';
      if (this.isValid()) {
        axios
            .post("http://0.0.0.0:8080/odu", {
              equation_id: this.equation_id,
              x0: this.x0,
              y0: this.y0,
              xn: this.xn,
              h: this.step,
              eps: this.accuracy
            })
            .then(response => {
              this.exact = response.data.exact
              this.runge = response.data.runge
              this.miln = response.data.miln
              this.euler = response.data.euler
              console.log(response.data);
              this.renderPlot();
            })
            .catch(() => {
              this.errorMessage = 'Error';
            });
      }
    },
    renderPlot() {
      const trace = {
        x: this.euler.map(euler => euler.x),
        y: this.euler.map(euler => euler.y),
        name: 'Euler'
      };

      const trace1 = {
        x: this.runge.map(runge => runge.x),
        y: this.runge.map(runge => runge.y),
        name: 'Runge'
      }

      const trace2 = {
        x: this.miln.map(miln => miln.x),
        y: this.miln.map(miln => miln.y),
        name: 'Miln'
      }

      const trace3 = {
        x: this.exact.map(exact => exact.x),
        y: this.exact.map(exact => exact.y),
        mode: 'markers',
        name: 'Points'
      };

      const layout = {
        xaxis: {
          title: 'x'
        },
        yaxis: {
          title: 'y',
          range: [-100, 100]
        }
      };

      Plotly.newPlot(this.$refs.plot, [trace, trace1, trace2, trace3], layout);
    }
  },
};
</script>

<style scoped>
.container {
  max-width: 800px;
  margin: 0 auto;
  font-family: Arial, sans-serif;
}

.buttons-container {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: 1rem;
}

.send-btn {
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 0.25rem;
  font-size: 1rem;
  font-weight: bold;
  color: #fff;
  background-color: #4caf50;
  cursor: pointer;
  transition: background-color 0.2s ease-in-out;
}

.send-btn:disabled {
  opacity: 0.5;
  cursor: default;
}

.send-btn {
  background-color: #2196f3;
}

.send-btn:hover {
  background-color: #0673c0;
}

.error-message {
  margin-top: 1rem;
  padding: 0.5rem;
  border: 1px solid #f44336;
  background-color: #ffebee;
  color: #f44336;
  font-size: 0.8rem;
  text-align: center;
  width: 100%;
}

input::-webkit-outer-spin-button,
input::-webkit-inner-spin-button {
  /* display: none; <- Crashes Chrome on hover */
  -webkit-appearance: none;
  margin: 0; /* <-- Apparently some margin are still there even though it's hidden */
}

.file-upload input[type="file"] {
  display: none;
}

.function-form {
  max-width: 400px;
  margin: 0 auto;
}

.form-group {
  margin-bottom: 20px;
}

label {
  font-weight: bold;
}

.form-control {
  width: 100%;
  padding: 8px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

</style>
