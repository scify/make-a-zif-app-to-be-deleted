<script>
import UnitAngstrom from "./units/UnitAngstrom.vue";
import UnitGMol from "./units/UnitGMol.vue";

export default {
  components: {
    UnitAngstrom,
    UnitGMol,
  },
  props: {
    ["selectedScenario"]: { required: true },
    ["listOfGases"]: { required: true },
  },
  emits: ["update:selectedGas"],
  created() {},
  data() {
    return {
      // Cosmetic thingies.
      hoverColumns: {
        gas: false,
      },
      inputGas: false,
      defaultGasModalDescription: "Select a gas from the following list",
      gasModalDescription: "",
    };
  },
  mounted() {
    this.gasModalDescription = this.defaultGasModalDescription;
    console.log("T2 acquired Gas: " + this.selectedScenario.gas);
    this.inputGas = this.selectedScenario.gas;
    document
      .getElementById("gasModal")
      .addEventListener("hide.bs.modal", this.onGasModalHide);
  },
  methods: {
    // Cosmetic methods.
    onMouseHover(columnGroup) {
      this.hoverColumns[columnGroup] = true;
    },
    onMouseLeave(columnGroup) {
      this.hoverColumns[columnGroup] = false;
    },
    clickGasRow(gas) {
      this.inputGas = gas;
    },
    onGasModalHide(e) {
      // Modal hiding is intercepted to enforce a Gas selection by the user,
      // even though the app was initially designed to allow null gas input.
      // As a default gas can't be proposed, we force the user to decide...
      if (this.inputGas === false || this.inputGas === "") {
        e.preventDefault();
        document
          .getElementById("gasModal")
          .classList.add("modal-static", "warning");
        setTimeout(() => {
          document.getElementById("gasModal").classList.remove("modal-static");
        }, 500);
        return false;
      }
      this.gasModalDescription = this.defaultGasModalDescription;
    },
    // Functional emitting.
    emitGas() {
      if (this.inputGas === "") {
        this.inputGas = false;
      }
      console.log("T2 emitted Gas update: ", this.inputGas);
      this.$emit("update:selectedGas", this.inputGas);
    },
  },
  watch: {
    inputGas() {
      if (this.inputGas === "" || this.inputGas === false) {
        this.gasModalDescription = "Select a gas and click the Add Gas button";
      } else {
        this.gasModalDescription =
          "Confirm selection by clicking the Add Gas button";
      }
      // Partially resets modal to default state if a gas is selected.
      const gasModal = document.getElementById("gasModal");
      const executeButton = document.getElementById("mazExecuteButton");
      executeButton.classList.remove("disabled");
      if (gasModal.classList.contains("warning")) {
        gasModal.classList.remove("warning");
      }
    },
  },
};
</script>

<template>
  <!-- v-if starts here -->
  <div class="maz-table pb-2" v-if="selectedScenario && listOfGases">
    <h4 class="two">Add a gas element</h4>
    <h5>Click on the Gas column to select a fume.</h5>
    <table class="table-multi limit-width">
      <colgroup class="header"></colgroup>
      <colgroup class="gas"></colgroup>
      <thead>
        <tr class="modal-row">
          <th
            scope="col"
            class="hidden limit-width"
            aria-label="Properties"
            role="columnheader"
          >
            <div class="col-modal">#</div>
          </th>
          <th scope="col" class="gas divider">
            <button
              id="gasModalBtn"
              class="col-modal"
              data-bs-toggle="modal"
              data-bs-target="#gasModal"
              v-on:mouseover="onMouseHover('gas')"
              v-on:mouseleave="onMouseLeave('gas')"
              aria-label="Selection of Gas"
              role="button"
            >
              <span class="col-title--arrow">Gas</span>
            </button>
          </th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th scope="row" class="text-start">Name</th>
          <td
            v-if="selectedScenario.gas"
            class="gas element"
            :class="hoverColumns.gas ? 'hover' : ''"
            v-html="selectedScenario.gas.name"
          ></td>
          <td v-else class="gas" :class="hoverColumns.gas ? 'hover' : 'faded'">
            &#8212;
          </td>
        </tr>
        <tr>
          <th scope="row" class="text-start">Mass <UnitGMol :abbr="true" /></th>
          <td
            v-if="selectedScenario.gas"
            class="gas"
            :class="hoverColumns.gas ? 'hover' : ''"
          >
            {{ selectedScenario.gas.mass.value }}
          </td>
          <td v-else class="gas" :class="hoverColumns.gas ? 'hover' : 'faded'">
            &#8212;
          </td>
        </tr>
        <tr>
          <th scope="row" class="text-start">
            Kinetic diameter <UnitAngstrom :abbr="true" />
          </th>
          <td
            v-if="selectedScenario.gas"
            class="gas"
            :class="hoverColumns.gas ? 'hover' : ''"
          >
            {{ selectedScenario.gas.kineticDiameter.value }}
          </td>
          <td v-else class="gas" :class="hoverColumns.gas ? 'hover' : 'faded'">
            &#8212;
          </td>
        </tr>
      </tbody>
    </table>
    <!-- Gas modal starts here -->
    <div
      class="modal fade"
      id="gasModal"
      v-bind:data-selectedGas="inputGas"
      data-bs-backdrop="static"
      tabindex="-1"
      aria-labelledby="gasModalLabel"
      aria-hidden="true"
    >
      <div class="modal-dialog modal-dialog-centered modal-dialog-scrollable">
        <div class="modal-content" id="gasModalContent">
          <div class="modal-header align-items-baseline">
            <div class="modal-title">
              <h6 id="gasModalLabel" class="fs-4 modal-label">Select a Gas</h6>
              <p id="gasModalDescription">
                {{ gasModalDescription }}
              </p>
              <span class="small">
                Keep in mind that the larger the gas is, the larger the
                modification impact will be on its diffusivity.
              </span>
            </div>
            <button
              type="button"
              id="gasModalXButton"
              class="btn-close"
              data-bs-dismiss="modal"
              aria-label="Close"
            ></button>
          </div>
          <div class="modal-body">
            <form @submit.prevent="emitGas()">
              <table class="table--select">
                <thead>
                  <tr>
                    <th scope="col">#</th>
                    <th scope="col">Gas</th>
                    <th scope="col">Kinetic diameter</th>
                    <th scope="col">Mass</th>
                  </tr>
                </thead>
                <tbody>
                  <tr @click="clickGasRow(false)">
                    <td>&#8212;</td>
                    <th scope="row" class="text-start">
                      <input
                        v-if="inputGas === false"
                        type="radio"
                        name="gas"
                        value=""
                        id="gasFalse"
                        checked="checked"
                        v-model="inputGas"
                      />
                      <input
                        v-else
                        type="radio"
                        name="gas"
                        value=""
                        id="gasFalse"
                        v-model="inputGas"
                      />
                      <label class="element" for="gasFalse">None</label>
                    </th>
                    <td class="text-center">&#8212;</td>
                    <td class="text-center">&#8212;</td>
                  </tr>
                  <tr
                    v-for="(gas, index) in listOfGases"
                    :key="index"
                    v-bind:title="gas.title"
                    @click="clickGasRow(gas)"
                  >
                    <td>{{ index + 1 }}</td>
                    <th scope="row" class="text-start">
                      <input
                        type="radio"
                        name="gas"
                        v-bind:value="gas"
                        v-bind:id="gas.key"
                        v-model="inputGas"
                      />
                      <label
                        class="element"
                        v-bind:for="gas.key"
                        v-html="gas.name"
                      >
                      </label>
                    </th>
                    <td class="text-center">
                      {{ gas.kineticDiameter.value
                      }}<span class="unit inline">{{
                        gas.kineticDiameter.unit.symbol
                      }}</span>
                    </td>
                    <td class="text-center">
                      {{ gas.mass.value
                      }}<span class="unit inline shrunk">{{
                        gas.mass.unit.symbol
                      }}</span>
                    </td>
                  </tr>
                </tbody>
              </table>
              <button
                type="submit"
                id="addGasButton"
                class="btn primary rounded me-auto"
                data-bs-dismiss="modal"
              >
                Add Gas
              </button>
            </form>
          </div>
        </div>
      </div>
    </div>
    <!-- Gas modal ends here-->
  </div>
  <!-- v-if ends here -->
</template>
