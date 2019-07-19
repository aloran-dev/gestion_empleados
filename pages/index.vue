<template>
  <v-container fluid>

    <v-toolbar
      flat
      color="white"
    >
      <v-toolbar-title>Lista de empleados</v-toolbar-title>
      <v-divider
        class="mx-2"
        inset
        vertical
      ></v-divider>
      <v-spacer></v-spacer>
      <v-dialog
        v-model="dialog"
        max-width="500px"
      >
        <template v-slot:activator="{ on }">
          <v-btn
            color="primary"
            dark
            class="mb-2"
            v-on="on"
          >Agregar empleado</v-btn>
        </template>

        <v-card>
          <v-card-title>
            <span class="headline">{{ formTitle }}</span>
          </v-card-title>

          <v-card-text>
            <v-container grid-list-md>
              <v-form
                ref="form"
                v-model="valid"
                lazy-validation
              >
                <v-layout wrap>
                  <v-flex xs12>
                    <v-text-field
                      v-model="editedItem.employee_name"
                      label="Nombre"
                      :rules="nameRules"
                      :counter="20"
                      required
                    ></v-text-field>
                  </v-flex>
                  <v-flex xs6>
                    <v-text-field
                      v-model="editedItem.employee_salary"
                      :rules="salaryRules"
                      label="Salario"
                    ></v-text-field>
                  </v-flex>
                  <v-flex xs6>
                    <v-text-field
                      v-model="editedItem.employee_age"
                      :rules="ageRules"
                      label="Edad"
                    ></v-text-field>
                  </v-flex>
                </v-layout>
              </v-form>
            </v-container>
          </v-card-text>

          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn
              color="blue darken-1"
              flat
              @click="close"
            >Cancelar</v-btn>
            <v-btn
              color="blue darken-1 white--text"
              @click="save"
              :disabled="!valid"
            >Guardar</v-btn>
          </v-card-actions>
        </v-card>

      </v-dialog>
    </v-toolbar>

    <v-data-table
      :headers="headers"
      :items="employees"
      class="elevation-1"
    >
      <template v-slot:items="props">
        <td>{{ props.item.id }}</td>
        <td>{{ props.item.employee_name }}</td>
        <td>{{ props.item.employee_salary }}</td>
        <td>{{ props.item.employee_age }}</td>
        <td class="justify-center layout px-0">
          <v-icon
            small
            class="mr-2"
            @click="editItem(props.item)"
          >
            edit
          </v-icon>
          <v-icon
            small
            @click="deleteItem(props.item)"
          >
            delete
          </v-icon>
        </td>
      </template>
    </v-data-table>

    <v-snackbar
      v-model="snackbar"
      :color="color"
      :multi-line="mode === 'multi-line'"
      :timeout="timeout"
      :vertical="mode === 'vertical'"
    >
      {{ text }}
      <v-btn
        dark
        flat
        @click="snackbar = false"
      >
        Close
      </v-btn>
    </v-snackbar>

  </v-container>
</template>

<script>
  import axios from 'axios'

  export default {
    async asyncData () {
      const { data } = await axios.get('http://dummy.restapiexample.com/api/v1/employees')
      return {
        dialog: false,
        valid: false,
        headers: [
          { text: 'ID', value: 'id' },
          { text: 'Nombre', value: 'employee_name' },
          { text: 'Salario', value: 'employee_salary' },
          { text: 'Edad', value: 'employee_age' },
          { text: 'Acciones', value: 'name', sortable: false }
        ],
        employees: data,
        editedIndex: -1,
        editedItem: {
          id: '',
          employee_name: '',
          employee_salary: '',
          employee_age: ''
        },
        defaultItem: {
          id: '',
          employee_name: '',
          employee_salary: '',
          employee_age: ''
        },
        nameRules: [
          v => !!v || 'Este campo es requerido',
          v => (/^[a-zA-Z]+$/.test(v)) || 'Tu nombre no debe contener numeros ni espacios',
          v => (v && v.length <= 20) || 'Tu nombre no debe contener más de 20 caracteres',
          v => (v && v.length >= 8) || 'Tu nombre debe contener más de 8 caracteres'
        ],
        salaryRules: [
          v => !!v || 'Este campo es requerido',
          v => (/^[0-9]+([.][0-9]+)?/.test(v)) || 'Por favor ingresa solo numeros sin espacios',
          v => (v && v <= 20000) || 'Tu salario no debe ser mayor a 20000',
          v => (v && v >= 8000) || 'Tu salario debe ser mayor a 8000'
        ],
        ageRules: [
          v => !!v || 'Este campo es requerido',
          v => (/^[0-9]+$/.test(v)) || 'Por favor ingresa solo numeros sin espacios',
          v => (v && v <= 35) || 'Tu edad no puede ser mayor a 35',
          v => (v && v >= 18) || 'Debes tener más de 18 años para continuar'
        ],
        snackbar: false,
        color: '',
        mode: '',
        timeout: 6000,
        text: ''
      }
    },

    computed: {
      formTitle () {
        return this.editedIndex === -1 ? 'Agregar nuevo empleado' : 'Editar empleado'
      }
    },

    watch: {
      dialog (val) {
        val || this.close()
      }
    },

    methods: {
      editItem (item) {
        this.editedIndex = this.employees.indexOf(item)
        this.editedItem = Object.assign({}, item)
        this.dialog = true
      },

      deleteItem (item) {
        const index = this.employees.indexOf(item)
        this.employees.splice(index, 1)
        axios.delete(`http://dummy.restapiexample.com/api/v1/delete/${item.id}`)
        this.text = `empleado ${item.employee_name} ¡eliminado!`
        this.color = 'success'
        this.snackbar = true
      },

      close () {
        this.dialog = false
        setTimeout(() => {
          this.editedItem = Object.assign({}, this.defaultItem)
          this.editedIndex = -1
        }, 300)
      },

      save () {
        if (this.$refs.form.validate()) {
          if (this.editedIndex === -1) {
            axios.post('http://dummy.restapiexample.com/api/v1/create', {
              "name": this.editedItem.employee_name,
              "salary": this.editedItem.employee_salary,
              "age": this.editedItem.employee_age,
            })
              .then(res => {
                this.text = `¡empleado ${res.data.name} creado!`
                this.color = 'success'
                this.snackbar = true
                this.employees.push({
                  "id": res.data.id,
                  "employee_name": res.data.name,
                  "employee_salary": res.data.salary,
                  "employee_age": res.data.age
                })
              })
          } else {
            axios.put(`http://dummy.restapiexample.com/api/v1/update/${this.editedItem.id}`, {
              "name": this.editedItem.employee_name,
              "salary": this.editedItem.employee_salary,
              "age": this.editedItem.employee_age,
            })
              .then(res => {
                this.text = '¡empleado actualizado!'
                this.color = 'success'
                this.snackbar = true
              })
            this.employees.splice(this.editedIndex, 1, this.editedItem)
          }

          this.close()
        }
      }
    }
  }
</script>
