<template>
  <div>
    
    <form @submit.prevent="agregar" v-if="!editarActivo">
      <h3>Agregar Notas</h3>
      <input type="text" placeholder="Nombre" class="form-control mb-2" v-model="nota.nombre">
      <input type="text" placeholder="Descripción" class="form-control mb-2" v-model="nota.descripcion">
      <button class="btn btn-primary" type="submit">Agregar</button>
    </form>

    <form @submit.prevent="editarNota(nota)" v-else>
      <h3>Editar Notas</h3>
      <input type="text" placeholder="Nombre" class="form-control mb-2" v-model="nota.nombre">
      <input type="text" placeholder="Descripción" class="form-control mb-2" v-model="nota.descripcion">
      <button class="btn btn-success" type="submit">Guardar</button>
      <button class="btn btn-danger" @click="cancelarEdicion">Cancelar</button>
    </form>
   
   <hr class="mt-3">
    <h3>Listado de notas</h3>
    <ul class="list-group my-2">
      <li class="list-group-item" v-for="(item, index) in notas" :key="index">
        <span class="badge badge-primary float-right">
          {{  getDateBeauty(item.updated_at)  }}
        </span>
        <p>{{ item.nombre }}</p> 
        <p>{{ item.descripcion }}</p>
        <button class="btn btn-danger btn-sm" @click="eliminarNota(item, index)">Eliminar</button>
        <button class="btn btn-warning btn-sm" @click="editarFormulario(item)">Editar</button>
      </li>
    </ul>
  </div>
</template>

<script>
import moment from 'moment';
export default {
  data(){
    return {
      formatFullDate: 'MM/DD/YYYY hh:mm',
      notas: [],
      nota: {nombre: '', descripcion: ''},
      editarActivo: false,
    }
  },
  created(){
    axios.get('/notas')
    .then(res => {
      this.notas = res.data;
    });
  },
  methods:{
    agregar(){
       if(this.nota.nombre.trim() === '' || this.nota.descripcion.trim() === ''){
        alert('Debes completar todos los campos antes de guardar');
        return;
      }
      console.log(this.nota.nombre, this.nota.descripcion);
      const params = {
        nombre: this.nota.nombre,
        descripcion: this.nota.descripcion
      };
      this.nota = {nombre: '', descripcion: ''}; 
      axios.post('/notas', params)
      .then( res => {
        this.notas.push(res.data);
      });
    },
    getDateBeauty(value){
      return moment(String(value)).format(this.formatFullDate);
    },
    eliminarNota(item, index){
      axios.delete(`/notas/${item.id}`)
      .then(() => {
        this.notas.splice(index, 1);
      });
    },
    editarFormulario(item){
      this.editarActivo = true;
      this.nota.nombre = item.nombre;
      this.nota.descripcion = item.descripcion;
      this.nota.id = item.id;
    },
    editarNota(item){
      const params = {
        nombre: item.nombre,
        descripcion: item.descripcion
      }
      axios.put(`/notas/${item.id}`, params)
      .then(res => {
        const index = this.notas.findIndex(nota => nota.id === res.data.id);
        this.notas[index] = res.data;

        this.editarActivo = false;
        this.nota.nombre = '';
        this.nota.descripcion = '';
      });
    },
    cancelarEdicion(){
      this.editarActivo = false;
      this.nota.nombre = '';
      this.nota.descripcion = '';
    }
  }
}
</script>