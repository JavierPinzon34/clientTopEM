<template>
  <div>
    <b-card>
      <h6> {{ id }}. {{ name }}</h6>
      <strong>${{ value }}</strong><span>({{ quantity }})</span>
    </b-card>
  </div>
</template>
<script>

export default {
  name: "index",
  props: {
    name: String,
    value: Number,
    quantity: Number,
    id: Number,
  },
  data() {
    return {}
  },
  methods: {
    onSubmit(event) {
      event.preventDefault();
      let me = this;
      this.axios
        .get("data", {
          params: me.form,
        })
        .then((response) => {
          me.items.push(response.data)
          if (response.data.length > 0) {
            this.$swal({
              toast: true,
              position: "top-end",
              icon: "success",
              title: "Exitoso con resultados",
              showConfirmButton: false,
              timer: 2000,
            });
          } else {
            this.$swal({
              toast: true,
              position: "top-end",
              icon: "warning",
              title: "Exitoso sin resultados",
              showConfirmButton: false,
              timer: 2000,
            });
          }
        });
    },
    logout() {
      localStorage.removeItem("token");
      this.$router.push("/login");
    },
  },
  computed: {
    rows() {
      return this.items.length;
    },
  },
};
</script>