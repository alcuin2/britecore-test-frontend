<template>
  <div>
    <b-container>
      <b-row class="justify-content-md-center mt-4 mb-4">
        <b-col col md="8">
          <b-card
            header="BriteCore Risks - Test"
            header-bg-variant="primary"
            header-text-variant="white"
          >
            <b-form @submit="onSubmit">
              <b-form-group label="Select Risk">
                <b-form-select
                  v-model="selected"
                  :options="risks"
                  @change="onSelected"
                ></b-form-select>
              </b-form-group>
              <div v-if="selected">
                <b-form-group label="Risk Name">
                  <b-form-input :value="selected_risk.risk_name" readonly />
                </b-form-group>
                <b-form-group label="Description">
                  <b-form-textarea
                    :value="selected_risk.description"
                    rows="3"
                    max-rows="6"
                    readonly
                  >
                  </b-form-textarea>
                </b-form-group>
                <b-form-group label="Insurer">
                  <b-form-input :value="selected_risk.insurer_name" readonly />
                </b-form-group>
                <b-form-group label="Fields">
                  <div v-for="field in selected_risk.fields" :key="field.uid">
                    <div v-if="field.field_type == 'text'">
                      <b-form-group :label="field.field_name">
                        <b-form-input required />
                      </b-form-group>
                    </div>
                    <div v-if="field.field_type == 'number'">
                      <b-form-group :label="field.field_name">
                        <b-form-input type="number" required />
                      </b-form-group>
                    </div>
                    <div v-if="field.field_type == 'date'">
                      <b-form-group :label="field.field_name">
                        <b-form-datepicker requied />
                      </b-form-group>
                    </div>
                    <div v-if="field.field_type == 'enum'">
                      <b-form-group :label="field.field_name">
                        <b-form-select
                          :options="field.enum_constants.split(',')"
                          required
                        />
                      </b-form-group>
                    </div>
                  </div>
                  <b-link v-b-modal.modal-1>Add a new field</b-link>
                </b-form-group>
                <b-form-group label="Risk Owner">
                  <b-form-input v-model="owner" required />
                </b-form-group>
                <b-form-group>
                  <b-button type="submit" variant="outline-primary"
                    >Create Risk</b-button
                  >
                </b-form-group>
              </div>
            </b-form>
          </b-card>
        </b-col>
      </b-row>
      <b-modal id="modal-1" title="Add a new risk field" @ok="handleOk">
        <b-alert show variant="danger" v-if="error != ''">{{ error }}</b-alert>
        <b-form-group label="Field Name">
          <b-form-input v-model="new_field.field_name" required />
        </b-form-group>
        <b-form-group label="Field Type">
          <b-form-select
            v-model="new_field.field_type"
            :options="['text', 'date', 'number', 'enum']"
            required
          ></b-form-select>
        </b-form-group>
        <b-form-group
          label="Field constants"
          v-if="new_field.field_type == 'enum'"
        >
          <b-form-input
            v-model="new_field.enum_constants"
            placeholder="Constants separated by commas"
            required
          />
        </b-form-group>
        <b-form-group label="Description">
          <b-form-input v-model="new_field.description" />
        </b-form-group>
      </b-modal>
    </b-container>
  </div>
</template>

<script>
import axios from "axios";

const url = "https://u7mkplvlga.execute-api.us-west-2.amazonaws.com/dev/";
export default {
  name: "Form",
  data: function () {
    return {
      selected: null,
      risks: [{ value: null, text: "Please select an option" }],
      selected_risk: "",
      owner: "",
      new_field: {
        field_name: "",
        field_type: "",
        enum_constants: "",
        description: "",
      },
      error: "",
    };
  },
  mounted: async function () {
    let response = await axios.get(url + "api/risktype/");
    response.data.forEach((elem) => {
      var risk = {
        value: elem.uid,
        text: `${elem.risk_name} | ${elem.insurer_name}`,
      };
      this.risks.push(risk);
    });
  },
  methods: {
    onSelected: async function () {
      let response = await axios.get(
        url + "api/risktype/" + this.selected + "/"
      );
      this.selected_risk = response.data;
    },
    handleOk: async function (bvModalEvt) {
      bvModalEvt.preventDefault();
      let payload = JSON.stringify({
        field_name: this.new_field.field_name,
        field_type: this.new_field.field_type,
        enum_constants: this.new_field.enum_constants,
        description: this.new_field.description,
        risk_type: this.selected,
      });
      try {
        let response = await axios.post(url + "api/riskfield/", payload, {
          headers: {
            "Content-Type": "application/json",
          },
        });
        if (response.status == 201) {
          this.selected_risk.fields.push(response.data);

          this.$bvModal.hide("modal-1");
        }
      } catch (err) {
        this.error = err.response.data.detail;
      }
      this.new_field.risk_type = "";
      this.new_field.field_name = "";
      this.new_field.enum_constants = "";
      this.new_field.description = "";
    },
    onSubmit: function (event) {
      event.preventDefault();
      // TODO: implement submit to create a risk instance for a user.
    },
  },
};
</script>

