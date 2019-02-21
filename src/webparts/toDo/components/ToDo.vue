<template>
    <div>
      <h1>Allgeier ToDo-List</h1>
      <ul class="list-group">
        <li v-for="item in items" v-bind:key="item" class="list-group-item">
          {{item.Value}}
          <button v-on:click="deleteItem(item.ID)" type="button" class="btn btn-default btn-sm" style="float: right;">
            <span class="glyphicon glyphicon-trash"></span> 
          </button>
        </li>
      </ul>
      <div class="row">
        <div class="col-lg-6">
          <div class="input-group">
            <input v-model="newItem" type="text" class="form-control">
            <span class="input-group-btn"><!-- Append button addon using class input-group-lg -->
              <button v-on:click="addItem(newItem)" class="btn btn-default" type="button">Add</button>
            </span>
          </div>
        </div>
      </div>
    </div>
</template>

<script lang="ts">
import { Vue, Component, Prop } from 'vue-property-decorator';

/**
 * Component's properties
 */
export interface IToDoProps {
    description: string;
}

import { sp } from "@pnp/sp";
import { SPComponentLoader } from '@microsoft/sp-loader';
import 'jquery';
require('bootstrap');

/**
 * Class-component
 */
@Component
export default class ToDo extends Vue implements IToDoProps {

    /**
     * implementing ISimpleWebPartProps interface
     */
    @Prop()
    public description: string;

    /**
     * Initialize ToDoList
     */
    constructor() {
      super();
      let cssURL = "https://stackpath.bootstrapcdn.com/bootstrap/3.4.1/css/bootstrap.min.css";
      SPComponentLoader.loadCss(cssURL);

      this.loadListItems();
    }

    public items: {ID: number, Value: string}[] = [];

    // Add new item to the list
    public addItem(value: string): void {
      sp.web.lists.getByTitle("ToDoList").items.add({Value: value}).then(data => {
          var item = {ID: data.data['Id'], Value: value};
          this.items.push(item);
        }).catch(error => {
            console.log('Error while adding a new ToDoItem. Error: ' + error);
      });
    }

    // Delete item by id
    public deleteItem(ID: number): void {
      this.items = this.items.filter(obj => obj.ID !== ID);

      sp.web.lists.getByTitle("ToDoList").items.getById(ID).delete().catch(error => {
          console.log('Error while deleting ToDoListitem. Error: ' + error);
      });
    }

    // Load all items
    public loadListItems(): void {
      sp.web.lists.getByTitle("ToDoList").items.get().then(data => {
        data.forEach(el => {
          this.items.push({ID: el.ID, Value: el.Value});
        })
      }).catch(error => {
        console.log("Error while loading data from ToDoList. Error: " + error);
      })
    }
}
</script>

<style lang="scss" module>



</style>


