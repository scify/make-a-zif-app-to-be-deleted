<script>
import { nextTick } from "vue";
import IntroTab from "@/components/partials/IntroTab.vue";
import ExamplesTab from "@/components/partials/ExamplesTab.vue";

export default {
  components: {
    IntroTab,
    ExamplesTab,
  },
  props: {
    ["selectedScenario"]: { required: true },
    ["scenarioResults"]: { required: true },
    ["scenarioHistory"]: { required: false, default: false },
  },
  emits: [
    "do:saveScenario",
    "do:deleteScenario",
    "do:downloadScenario",
    "do:loadExampleScenario",
    "do:loadScenario",
  ],
  data() {
    return {
      inputScenarioName: null,
      countSavedScenarios: 0,
    };
  },
  mounted() {},
  methods: {
    parseMetal(key) {
      const metal = key
        ? this.$root["listOfMetals"].find((i) => i.key === key)
        : null;
      return metal ? metal.title : "None";
    },
    parseLinker(key) {
      const linker = key
        ? this.$root["listOfLinkers"].find((i) => i.key === key)
        : null;
      return linker ? linker.name : "None";
    },
    parseGroup(key) {
      const group = key
        ? this.$root["listOfGroups"].find((i) => i.key === key)
        : null;
      return group ? this.parseSubScript(group.name) : "None";
    },
    parseGas(key) {
      const gas = key
        ? this.$root["listOfGases"].find((i) => i.key === key)
        : null;
      return gas ? gas.title : "None";
    },
    loadExampleScenario(exampleScenario) {
      console.log("Maz Tabs received loadExampleScenario");
      this.$emit("do:loadExampleScenario", exampleScenario);
    },
    isRestored(scenario) {
      const selected = this.selectedScenario;
      const current = scenario.scenario;
      return (
        selected.gas.key === current.gas &&
        selected.metal.key === current.metal &&
        selected.funcGroup1.key === current.funcGroup1 &&
        selected.funcGroup2.key === current.funcGroup2 &&
        selected.funcGroup3.key === current.funcGroup3 &&
        selected.linker1.key === current.linker1 &&
        selected.linker2.key === current.linker2 &&
        selected.linker3.key === current.linker3
      );
    },

    /**
     * Parses <sub>digit</sub> and returns the corresponding JavaScript friendly
     * unicode character. E.g. <sub>2</sub> becomes \u2082.
     * @param unit A unit which might contain <sub>scripts.
     * @returns {string} The unit in plain unicode friendly text.
     */
    parseSubScript(unit) {
      return unit.replace(/<sub>(\d)<\/sub>/g, (match, digit) => {
        const unicodeCode = 0x2080 + parseInt(digit);
        return String.fromCharCode(unicodeCode);
      });
    },

    startDeleteScenario(scenarioDate, scenarioEl) {
      const zifItem = document.getElementById(scenarioEl);
      // Experimental Simplified View Transition.
      // @link https://developer.mozilla.org/en-US/docs/Web/API/Document/startViewTransition
      console.log("Start scenario removal...", scenarioEl);
      if (zifItem && document.startViewTransition) {
        document.startViewTransition(() => {
          zifItem.remove();
          nextTick(() => {
            this.$emit("do:deleteScenario", scenarioDate);
          });
        });
      } else {
        this.$emit("do:deleteScenario", scenarioDate);
      }
    },

    saveScenario() {
      let scenarioName = "Scenario";
      if (this.inputScenarioName) {
        scenarioName = this.inputScenarioName.trim();
        // Strips (supposedly) a lot of special characters:
        scenarioName = scenarioName.replace(
          /[`~!@#$%^&*()_|+\-=?;:'",.<>{}[\]\\/]/gi,
          ""
        );
        // Enforcing a 20-character limit as desktop view has limited width:
        if (scenarioName.length > 20) {
          scenarioName = scenarioName.substring(0, 20);
        }
      } else {
        // As the inputScenarioName no longer exists (probably) we have to
        // create a fancy name by ourselves like "Scenario n". For that, we
        // first have to count the amount of scenarios that the user has already
        // saved.
        if (this.scenarioHistory) {
          if (this.scenarioHistory.length) {
            this.countSavedScenarios = this.scenarioHistory.length;
            let tempScenarioName = `${scenarioName} ${
              this.countSavedScenarios + 1
            }`;
            // But, yes, that might be an issue if user has only a "Scenario 2"
            // saved & we are trying to save a 2nd one, which would have the
            // exact same name. So, let's just take the latest scenario...
            const existingScenario = this.scenarioHistory.find(
              (i) => i.name === tempScenarioName
            );
            let plusCount = 1;
            if (existingScenario) {
              plusCount++;
            }
            scenarioName = `${scenarioName} ${
              this.countSavedScenarios + plusCount
            }`;
          }
        } else {
          scenarioName = `${scenarioName} 1`;
        }
        this.countSavedScenarios++;
      }
      console.log("Trying to save Scenario with name: " + scenarioName);
      // Experimental View Transition (read previous, related comment).
      if (document.startViewTransition) {
        document.startViewTransition(() => {
          this.$emit("do:saveScenario", scenarioName);
        });
      } else {
        this.$emit("do:saveScenario", scenarioName);
      }
    },
  },
  watch: {
    scenarioResults() {
      if (
        this.scenarioResults.scenario &&
        Object.keys(this.scenarioResults.scenario).length > 0
      ) {
        if (this.scenarioResults.showDiff && this.scenarioResults.showSave) {
          nextTick(() => {
            document.querySelector("#generatedZif").classList.add("new");
            setTimeout(() => {
              document.querySelector("#generatedZif").classList.remove("new");
              document
                .querySelector("#mazExecuteButton")
                .classList.remove("disabled");
            }, 2500); // Adjust timing as needed
          });
        }
      }
    },
  },
};
</script>

<template>
  <div class="introduction mt-4 mb-1 pt-2 pb-0">
    <div class="maz-tabs" role="navigation">
      <ul id="mazTabs" class="nav nav-tabs" role="tablist">
        <li class="nav-item" role="presentation">
          <button
            class="nav-link active pt-3 px-md-5 px-sm-4 px-3"
            id="mazIntroTab"
            data-bs-toggle="tab"
            data-bs-target="#mazIntroPane"
            type="button"
            role="tab"
            aria-controls="mazIntroPane"
            aria-selected="true"
          >
            Intro
          </button>
        </li>
        <li class="nav-item" role="presentation">
          <button
            class="nav-link pt-3 px-sm-4 px-3"
            id="mazExamplesTab"
            data-bs-toggle="tab"
            data-bs-target="#mazExamplesPane"
            type="button"
            role="tab"
            aria-controls="mazExamplesPane"
            aria-selected="false"
          >
            Examples
          </button>
        </li>
        <li class="nav-item" role="presentation">
          <button
            class="nav-link pt-3 px-sm-4 px-3"
            id="mazHistoryTab"
            data-bs-toggle="tab"
            data-bs-target="#mazHistoryPane"
            type="button"
            role="tab"
            aria-controls="mazHistoryPane"
            aria-selected="false"
          >
            History
          </button>
        </li>
      </ul>
    </div>

    <div id="mazTabPanels" class="tab-content maz-tab-content">
      <div
        id="mazIntroPane"
        class="tab-pane fade show active"
        role="tabpanel"
        aria-labelledby="mazIntroTab"
        tabindex="0"
      >
        <IntroTab :selected-scenario="selectedScenario" />
      </div>
      <div
        id="mazExamplesPane"
        class="tab-pane fade"
        role="tabpanel"
        aria-labelledby="mazExamplesTab"
        tabindex="0"
      >
        <ExamplesTab @do:load-example-scenario="loadExampleScenario" />
      </div>
      <div
        id="mazHistoryPane"
        class="tab-pane fade"
        role="tabpanel"
        aria-labelledby="mazHistoryTab"
        tabindex="0"
      >
        <!-- no memo or history -->
        <template
          v-if="
            !scenarioResults.showDiff &&
            !scenarioResults.showStatus &&
            !scenarioHistory
          "
        >
          <div
            class="container-fluid mt-4 px-3 py-3 border border-accent documentation"
          >
            This panel will allow you to review the diffusivity and the
            properties of all the ZIFs you have designed and either download or
            restore them.
          </div>
        </template>
        <!-- /no memo or history -->
        <!-- memo &|| history -->
        <template v-else>
          <div class="container-fluid mt-4 px-0 py-2 log">
            <div
              class="zif-history zif-items row row-cols-xl-4 row-cols-md-3 row-cols-sm-2 row-cols-1 gx-lg-5 gx-md-2 gx-sm-4 gy-5 gx-0"
            >
              <!-- memo starts here -->
              <template
                v-if="
                  scenarioResults &&
                  (scenarioResults.showDiff || scenarioResults.showStatus)
                "
              >
                <!-- zif starts here -->
                <div
                  class="col zif-item zif-item--0"
                  style="{ 'view-transition-name': 'zif-item--0' }"
                >
                  <div
                    :class="[
                      'zif',
                      'memo',
                      { loading: scenarioResults.showStatus },
                      {
                        'd-none':
                          !scenarioResults.showDiff &&
                          !scenarioResults.showStatus,
                      },
                    ]"
                    id="generatedZif"
                  >
                    <div class="zif--header">
                      {{
                        scenarioResults.showDiff
                          ? "Generated ZIF diffusivity:"
                          : "Generating ZIF"
                      }}
                    </div>
                    <div class="zif--diffusivity">
                      <span v-if="scenarioResults.showDiff">
                        {{ scenarioResults.diffusion }}
                      </span>
                      <span v-if="scenarioResults.showStatus">
                        {{ scenarioResults.status }}
                      </span>
                    </div>
                    <div class="zif--units d-grid d-flex flex-wrap gap-0">
                      <div class="unit flex-grow-1">
                        <dl>
                          <dt>Metal</dt>
                          <dd class="text-capitalize">
                            {{
                              this.parseMetal(scenarioResults.scenario.metal)
                            }}
                          </dd>
                        </dl>
                      </div>
                      <div class="unit">
                        <dl>
                          <dt>Organic linkers</dt>
                          <dd>
                            {{
                              this.parseLinker(scenarioResults.scenario.linker1)
                            }}
                            {{
                              this.parseLinker(scenarioResults.scenario.linker2)
                            }}
                            {{
                              this.parseLinker(scenarioResults.scenario.linker3)
                            }}
                          </dd>
                        </dl>
                      </div>
                      <div class="unit flex-grow-1">
                        <dl>
                          <dt>Functional groups</dt>
                          <dd>
                            {{
                              this.parseGroup(
                                scenarioResults.scenario.funcGroup1
                              )
                            }}
                            {{
                              this.parseGroup(
                                scenarioResults.scenario.funcGroup2
                              )
                            }}
                            {{
                              this.parseGroup(
                                scenarioResults.scenario.funcGroup3
                              )
                            }}
                          </dd>
                        </dl>
                      </div>
                      <div class="unit">
                        <dl>
                          <dt>Gas</dt>
                          <dd class="text-capitalize">
                            {{ this.parseGas(scenarioResults.scenario.gas) }}
                          </dd>
                        </dl>
                      </div>
                    </div>
                    <div class="zif--date">
                      Date:
                      <time v-bind:datetime="scenarioResults.date">{{
                        scenarioResults.formattedDate
                      }}</time>
                    </div>
                    <div class="zif--actions d-grid gap-2 d-md-flex">
                      <button
                        v-if="scenarioResults.showDiff"
                        :class="[
                          'btn',
                          'btn-sm',
                          'btn-primary',
                          'flex-fill',
                          { disabled: !scenarioResults.showSave },
                        ]"
                        :disabled="!scenarioResults.showSave"
                        @click="saveScenario"
                        class="btn btn-sm btn-primary flex-fill"
                        data-bs-toggle="tooltip"
                        data-bs-placement="top"
                        aria-label="Save ZIF model to your browser's local cache history"
                        title="Save this ZIF model to your browser's history"
                        type="button"
                      >
                        {{ scenarioResults.showSave ? "Save" : "Saved!" }}
                      </button>
                      <button
                        v-if="!scenarioResults.showDiff"
                        class="btn btn-sm btn-primary disabled flex-fill"
                        disabled
                      >
                        ...
                      </button>
                    </div>
                  </div>
                </div>
                <!-- /zif ends here -->
              </template>
              <!-- /memo ends here -->

              <!-- history starts here -->
              <template v-if="scenarioHistory">
                <template
                  v-for="(scenario, key) in scenarioHistory.slice().reverse()"
                  :key="scenario.dateTimeStamp"
                >
                  <!-- zif starts here -->
                  <div
                    :class="[
                      'col',
                      'zif-item',
                      'zif-item--' + scenario.dateTimeStamp,
                    ]"
                    :style="{
                      'view-transition-name':
                        'zif-item-' + scenario.dateTimeStamp,
                    }"
                    :id="`zif-item-${scenario.dateTimeStamp}`"
                  >
                    <div
                      :class="[
                        'zif',
                        'scenario',
                        { restored: isRestored(scenario) },
                      ]"
                      :id="`saved-scenario-${key}`"
                    >
                      <div class="zif--header">
                        ZIF {{ scenario.name }} diffusivity:
                      </div>
                      <div class="zif--diffusivity">
                        {{ scenario.diffusion }}
                      </div>
                      <div class="zif--units d-grid d-flex flex-wrap gap-0">
                        <div class="unit flex-grow-1">
                          <dl>
                            <dt>Metal</dt>
                            <dd class="text-capitalize">
                              {{ this.parseMetal(scenario.scenario.metal) }}
                            </dd>
                          </dl>
                        </div>
                        <div class="unit">
                          <dl>
                            <dt>Organic linkers</dt>
                            <dd>
                              {{ this.parseLinker(scenario.scenario.linker1) }}
                              {{ this.parseLinker(scenario.scenario.linker2) }}
                              {{ this.parseLinker(scenario.scenario.linker3) }}
                            </dd>
                          </dl>
                        </div>
                        <div class="unit flex-grow-1">
                          <dl>
                            <dt>Functional groups</dt>
                            <dd>
                              {{
                                this.parseGroup(scenario.scenario.funcGroup1)
                              }}
                              {{
                                this.parseGroup(scenario.scenario.funcGroup2)
                              }}
                              {{
                                this.parseGroup(scenario.scenario.funcGroup3)
                              }}
                            </dd>
                          </dl>
                        </div>
                        <div class="unit">
                          <dl>
                            <dt>Gas</dt>
                            <dd class="text-capitalize">
                              {{ this.parseGas(scenario.scenario.gas) }}
                            </dd>
                          </dl>
                        </div>
                      </div>
                      <div class="zif--date">
                        Date:
                        <time v-bind:datetime="scenario.date">{{
                          scenario.formattedDate
                        }}</time>
                      </div>
                      <div class="zif--actions d-grid gap-2 d-md-flex">
                        <button
                          :class="[
                            'btn',
                            'btn-sm',
                            'btn-primary',
                            'flex-fill',
                            'restore',
                            { disabled: isRestored(scenario) },
                          ]"
                          type="button"
                          aria-label="Restore ZIF Model"
                          @click="$emit('do:loadScenario', scenario.date)"
                        >
                          Restore
                        </button>
                        <button
                          class="btn btn-sm btn-primary"
                          type="button"
                          aria-label="Download ZIF model as a JSON file"
                          @click="$emit('do:downloadScenario', scenario.date)"
                        >
                          Download
                        </button>
                        <button
                          class="btn btn-sm btn-outline-primary"
                          type="button"
                          aria-label="Delete ZIF model"
                          @click="
                            startDeleteScenario(
                              scenario.date,
                              `zif-item-${scenario.dateTimeStamp}`
                            )
                          "
                        >
                          Delete
                        </button>
                      </div>
                    </div>
                  </div>
                  <!-- / zif ends here -->
                </template>
              </template>
              <!-- / history ends here -->
            </div>
          </div>
        </template>
        <!-- / memo &|| history -->
      </div>
    </div>
  </div>
</template>
