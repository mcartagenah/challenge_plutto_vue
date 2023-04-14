<script>
export default {
  methods: {
   // From: https://github.com/felipejoq/validar-rut-chileno/blob/master/js/script.js
    clean(rut) {
      return typeof rut === 'string'
        ? rut.replace(/^0+|[^0-9kK]+/g, '').toUpperCase()
        : ''
    },
    format(rut) {
      rut = this.clean(rut)

      var result = rut.slice(-4, -1) + '-' + rut.substr(rut.length - 1)
      for (var i = 4; i < rut.length; i += 3) {
        result = rut.slice(-3 - i, -i) + '.' + result
      }

      return result;
    },
    validRut(rut) {
      if (rut.length <= 1) {
        return false
      }

      // Obtiene el valor ingresado quitando puntos y guión.
      let valor = this.clean(rut);

      // Divide el valor ingresado en dígito verificador y resto del RUT.
      let bodyRut = valor.slice(0, -1);
      let dv = valor.slice(-1).toUpperCase();

      // Separa con un Guión el cuerpo del dígito verificador.
      rut = this.format(rut);

      // Si no cumple con el mínimo ej. (n.nnn.nnn)
      if (bodyRut.length < 7) {
        return false
      }

      // Calcular Dígito Verificador "Método del Módulo 11"
      let suma = 0;
      let multiplo = 2;

      // Para cada dígito del Cuerpo
      for (let i = 1; i <= bodyRut.length; i++) {
        // Obtener su Producto con el Múltiplo Correspondiente
        let index = multiplo * valor.charAt(bodyRut.length - i);

        // Sumar al Contador General
        suma = suma + index;

        // Consolidar Múltiplo dentro del rango [2,7]
        if (multiplo < 7) {
          multiplo = multiplo + 1;
        } else {
          multiplo = 2;
        }
      }

      // Calcular Dígito Verificador en base al Módulo 11
      let dvEsperado = 11 - (suma % 11);

      // Casos Especiales (0 y K)
      dv = dv == "K" ? 10 : dv;
      dv = dv == 0 ? 11 : dv;

      // Validar que el Cuerpo coincide con su Dígito Verificador
      if (dvEsperado != dv) {
        return false
      } else {
        return true;
      }
    },
    async checkTaxSituation() {
      this.responseData = ''

      const requestOptions = {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ company_id: this.inputValue })
      }

      const apiUrl = import.meta.env.VITE_API_URL;

      if (this.validRut(this.inputValue)) {
        this.responseData = 'Consulting . . .'
        const response = await fetch(apiUrl + "/tax_situation", requestOptions);
        const data = await response.json();
        this.responseData = data;
      } else {
        this.responseData = 'INVALID RUT'
      }
    },
  },
  data() {
    return {
      inputValue: '',
      responseData: '',
    }
  }
}
</script>

<template>
  <div class="input-card">
    <h2>Enter RUT:</h2>
    <input v-model="inputValue" type="text">
  </div>
  <div>
    <button @click=checkTaxSituation>Consultar</button>
  </div>

  <div class="response-card">
    <h2>Response:</h2>
    <div v-if="responseData !== ''">
      <h3>{{responseData.business_name}}</h3>
      <div class="activity-card">
        <h4>Activities</h4>
        <div v-for="item in responseData.activities" :key="item.code">
          <p>Date: {{item.date}}</p>
          <p>Code: {{item.code}}</p>
          <p>Name: {{item.name}}</p>
          <p>Exempt: {{item.exempt}}</p>
          <p>Category: {{item.category}}</p>
          <br/>
        </div>
      </div>
      <br/>
      <div class="tax-situation-card">
        <h4>Tax Situation</h4>
        <p>Is small enterprise: {{responseData.tax_situation.is_small_enterprise}}</p>
        <p>Can tax in foreign money: {{responseData.tax_situation.can_tax_in_foreign_money}}</p>
        <p>Activity initiation date: {{responseData.tax_situation.activity_initiation_date}}</p>
        <p>Has activity initiation: {{responseData.tax_situation.has_activity_initiation}}</p>
      </div>
    </div>
  </div>
</template>

<style scoped>
  /* TODO */
</style>