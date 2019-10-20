<template>
  <div id="app">
    <!-- NAVIGATION -->
    <nav class="navbar navbar-dark">
      <div class="col-sm-12 text-center">
        <img src="./assets/Ecolunch.png">
      </div>

    </nav>
    <!-- MAIN CONTENT -->
    <div class="container">
      <div class="row mt-2">
        <div class="col-sm-12 text-center">
          <div class="card">
            <div class="card-header">
              <h3>Agregar nuevo préstamo</h3>
            </div>
            <div class="card-body">
              <form @submit.prevent="addRegister">
                <div class="form-group">
                  <input
                    type="email"
                    class="form-control"
                    required name="correo"
                    v-model="newRegister.myi"
                    placeholder="Correo institucional"
                  >
                </div>
                <div class="form-group">
                  <input
                    type="text"
                    class="form-control"
                    required name="name"
                    v-model="newRegister.name"
                    placeholder="Nombre"
                  >
                </div>
                <div class="form-group">
                  <div class="row">
                    <div class="col-md-4">
                      <input
                        type="text"
                        class="form-control"
                        required name="codigo"
                        v-model="newRegister.Ctoper"
                        placeholder="Código tupper"
                      >
                    </div>
                    <div class="col-md-8">
                      <input
                        type="date"
                        class="form-control"
                        required name="fecha"
                        v-model="newRegister.tiempo"
                        placeholder="Fecha entrega"
                      >
                      <input
                        class="form-control"
                        required name="fecha"
                        v-model="newRegister.fechaEntrega"
                        hidden
                      >
                    </div>
                  </div>
                </div>
                <div class="form-group">
                  <select
                    id="inputState"
                    class="form-control"
                    required name="option"
                    v-model="newRegister.entregado"
                  >
                    <option value="" selected disabled>Elija una opción</option>
                    <option>Pendiente entregar</option>
                  </select>
                </div>
                <button type="submit" class="btn btn-primary">Guardar</button>
              </form>
            </div>
          </div>
        </div>
        <div class="row mt-2">
          <div class="col-12 text-center">
            <div class="card">
              <div class="card-header">
                <h3>Lista de prestamos</h3>
                <button v-on:click="FilterVal(2)" class="btn btn-success">Entregados</button>
                <button v-on:click="FilterVal(3)" class="btn btn-danger">No entregados</button>
                <button v-on:click="FilterVal(1)" class="btn btn-info">Mostrar todos</button>
              </div>
              <div class="card-body">
                <table class="table table-striped table-bordered">
                  <thead>
                    <tr>
                      <th>Correo institucional</th>
                      <th>Nombre</th>
                      <th>Código tupper</th>
                      <th>Día de préstamo</th>
                      <th>Día de entrega</th>
                      <th>Estado</th>
                      <th>Correos enviados</th>
                      <!--<th>Eliminar</th>-->
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="(w,i) in Register" :key="i">
                      <td v-show="Filter(w.entregado)">{{w.myi}}</td>
                      <td v-show="Filter(w.entregado)">{{w.name}}</td>
                      <td v-show="Filter(w.entregado)">{{w.Ctoper}}</td>
                      <td v-show="Filter(w.entregado)">{{w.fechaEntrega}}</td>
                      <td v-show="Filter(w.entregado)">{{w.tiempo}}</td>
                      <td  v-show="Filter(w.entregado)">{{w.entregado}}
                        <button
                          @click="updateRegister(w)"
                          class="btn btn-primary btn-sm"
                          :disabled="w.entregado=='ENTREGADO'"
                        >Entregado</button>
                      </td>

                      <td v-show="Filter(w.entregado)">
                        <!--{{
                        //5 horas es el limite primer aviso
                        (w.entregado!="ENTREGADO" && w.mails==0 && calculate(w.tiempo)[0]<=5)?SendMail(w,w.myi,w.Ctoper,w.mails+1,w.name,w.fechaEntrega):""
                        }}
                        {{
                        (w.entregado!="ENTREGADO" && w.mails==1 && calculate(w.tiempo)[0]<=0)?SendMail(w,w.myi,w.Ctoper,w.mails+1,w.name,w.fechaEntrega):""
                        }}--> 
                        <form @submit.prevent="SendMail(w,w.myi,w.Ctoper,w.mails+1,w.name,w.fechaEntrega)">
                          <button
                            type="submit"
                            class="btn btn-primary btn-lg"
                            :disabled="w.entregado=='ENTREGADO'"
                          >Enviar aviso #{{w.mails+1}}</button>
                        </form>
                      </td>
                      <!--<td>
                        <button @click="deleteRegister(w)" class="btn btn-danger">
                          Delete
                        </button>
                      </td>-->
                    </tr>
                  </tbody>
                </table>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>



<!--<script type="text/javascript">
     setInterval(function(tiempo){
        var deadline = new Date('02/22/2012 20:00 GMT-0800');
        var ahorita = new Date();
        var diff = deadline - ahorita;
        var diff_seg = Math.floor(diff / 1000);
        var seg = diff_seg % 60;
        var min = Math.floor(diff_seg / 60) % 60;
        var hr = Math.floor(diff_seg / 3600);
        document.getElementById('cosa').innerHTML = hr + ':' + min + ':' + seg;
     }, 1000);
   </script> -->



<script>
// Firebase
import Firebase from "firebase";
import config from "./config";
import moment from "moment";
import "moment-timezone";
import timediff from "timediff";
import axios from "axios";
let app = Firebase.initializeApp(config);
let db = app.database();
let RegisterRef = db.ref("Register");

// toast
import toastr from "toastr";
import Vue from "vue";

export default {
  name: "App",
  firebase: {
    Register: RegisterRef
  },
  data: {
    FilterType: 1
  },
  data() {
    return {
      newRegister: {
        myi: "",
        name: "",
        mails: 0,
        Ctoper: "",
        fechaEntrega: moment()
          .tz("America/Cancun")
          .format("YYYY-MM-DD"),
        tiempo: "",
        entregado: ""
      }
    };
  },
  methods: {
    addRegister() {
      RegisterRef.push(this.newRegister);
      this.newRegister.myi = "";
      this.newRegister.name = "";
      this.newRegister.Ctoper = "";
      this.newRegister.fechaEntrega = moment()
        .tz("America/Cancun")
        .format("YYYY-MM-DD");
      this.newRegister.tiempo = "";
      this.newRegister.entregado = "";
      this.newRegister.mails = 0;
    },
    deleteRegister(Register) {
      if (confirm("Seguro que desea elimnarlo")) {
        RegisterRef.child(Register[".key"]).remove();
        toastr.success("Registro Eliminado");
      }
    },
    updateRegister(Register) {
      if (confirm("¿Desa continuar con la modificacion?")) {
        RegisterRef.child(Register[".key"]).update({
          entregado: "ENTREGADO"
        });
        toastr.success("Register modificado");
      }
    },

    calculate(DeliverTime) {
      let after = moment(DeliverTime + "20:00:00", "YYYY-MM-DD HH:mm:ss");
      let now = moment().tz("America/Cancun");
      let difference = timediff(now.toDate(), after.toDate(), "HmS");
      let final;
      if (difference.hours <= 96) {
        final = [difference.hours, difference.minutes, difference.seconds];
      } else {
        final = [0, 0, 0];
      }
      return final;
    },
    SendMail(Register, mail, tupper, number, name, fechaEntrega) {
      const params = {
        Sender: mail,
        tupper: tupper,
        name: name,
        fechaEntrega: fechaEntrega,

      };
      axios.post("https://mailer-tupper.herokuapp.com/", params).then(res => {
        console.log(res.data);
      });
      RegisterRef.child(Register[".key"]).update({
        mails: number
      });
    },
    FilterVal: async function(value) {
      this.FilterType = value;
      await this.$forceUpdate();
    },
    Filter(Entregado) {
      switch (this.FilterType) {
        case 1:
          return 1;
          break;
        case 2:
          return Entregado == "ENTREGADO";
          break;
        case 3:
          return Entregado != "ENTREGADO";
          break;
        default:
          return true;
          break;
      }
    }
  }
};
</script>

<style>
#app {
  background: #ffefba; /* fallback for old browsers */
  background: -webkit-linear-gradient(
    to right,
    #ffffff,
    #ffefba
  ); /* Chrome 10-25, Safari 5.1-6 */
  background: linear-gradient(
    to right,
    #239B56,
    #239B56  
  ); /* W3C, IE 10+/ Edge, Firefox 16+, Chrome 26+, Opera 12+, Safari 7+ */
  height: 100%;
}
</style>
