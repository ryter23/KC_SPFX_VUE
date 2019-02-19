<template>
    <div>
      <h1>Allgeier ToDo-List</h1>
      <ul>
        <li v-for="item in items" v-bind:key="item">
          {{item.Value}} {{item.ID}} <button v-on:click="deleteItem(item.ID)">Delete</button>
        </li>
      </ul>
      <input type="text" v-model="newItem" />
      <button v-on:click="addItem(newItem)">Add</button>
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
@import '~@microsoft/sp-office-ui-fabric-core/dist/sass/_SPFabricCore.scss';

</style>


