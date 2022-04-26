<template>
  <div class="container text-center">
    <div class="card-login">      
      <div class="text-center">
        <img alt="Vue logo" src="../assets/logo.png">
      </div>
      <b-card>
        <b-form @submit="onSubmit" @reset="onReset" v-if="show">
          <b-form-group
            id="input-group-1"
            label="Correo electrónico:"
            label-for="input-1"
          >
            <b-form-input
              id="input-1"
              v-model="form.email"
              type="email"
              placeholder="Ingrese E-mail"
              required
            ></b-form-input>
          </b-form-group>

          <b-form-group id="input-group-2" label="Contraseña:" label-for="input-2">
            <b-form-input
              id="input-2"
              type="password"
              v-model="form.password"
              placeholder="********"
              required
            ></b-form-input>
          </b-form-group>

          <div class="text-center">
            <b-button type="submit" variant="info">Iniciar</b-button>
          </div>
        </b-form>
      </b-card>
    </div>
  </div>
</template>

<script>

  export default {
    data() {
      return {
        form: {
          email: 'jspinzonr@misena.edu.co',
          password: '123456789',
        },
        show: true
      }
    },
    methods: {
      onSubmit(event) {
        event.preventDefault()
        let me = this        
        this.axios.post('login', {          
            email: me.form.email,
            password: me.form.password
        })
        .then(res => {
          console.log(res)
          localStorage.token = res.data.access_token;
          this.$swal({
            position: 'top',
            icon: 'success',
            title: 'Bienvenido',
            showConfirmButton: false,
            timer: 1000
          });
          setTimeout(() => {
            this.$router.push('/')
          }, 1050);  
        })
        .catch(err => {
          if(err.response.status==401){
            this.$swal({
              position: 'top',
              icon: 'error',
              title: 'Error de credenciales',
              showConfirmButton: false,
              timer: 1000
            });
          }
        })
      },
      onReset(event) {
        event.preventDefault()
        // Reset our form values
        this.form.email = ''
        this.form.name = ''
        this.form.food = null
        this.form.checked = []
        // Trick to reset/clear native browser form validation state
        this.show = false
        this.$nextTick(() => {
          this.show = true
        })
      }
    }
  }
</script>
<style>
  .card-login {
    max-width: 300px;
    margin: auto;
    text-align: left;
  }
</style>