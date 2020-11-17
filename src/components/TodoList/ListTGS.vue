<template>
    <v-main class="list">
        <h3 class="text-h3 font-weight-medium mb-5">To Do List</h3>

        <v-card>
            <v-card-title>
                <v-text-field
                    v-model="search"
                    append-icon="mdi-magnify"
                    label="Search"
                    single-line
                    hide-details>
                </v-text-field>

                <v-spacer></v-spacer>

                <v-col
                    cols="12"
                    sm="5">
                    <v-select
                        v-model="urut"
                        :items="['Penting', 'Tidak penting']"
                        label="Urutkan"
                        @change="urutkan(urut)"
                        dense
                        outlined
                        hide-details>
                    </v-select>
                </v-col>

                <v-btn color="success" dark @click="dialog = true">
                    Tambah
                </v-btn>
            </v-card-title>

            <v-data-table 
            :headers="headers" 
            :items="todos" 
            :search="search"
            :single = "search"
            :expanded.sync = "expanded"
            item-key = "note"
            show-expand
            class = "elevation-1">

                <template v-slot:[`item.priority`]="{ item }">
                    <td>
                        <v-chip v-if="item.priority == 'Penting'" color="red" outlined label>
                            {{ item.priority }}
                        </v-chip>
                        <v-chip v-else-if="item.priority == 'Biasa'" color="blue" outlined label>
                            {{ item.priority }}
                        </v-chip>
                        <v-chip v-else color="green" outlined label>
                            {{ item.priority }}
                        </v-chip>
                    </td>
                </template>

                <template v-slot:[`item.checks`]="{ item }">
                    <v-checkbox multiple :key="item" @click.capture.stop="selected(item)"/>
                </template>

                <template v-slot:expanded-item="{ headers, item }">
                    <td :colspan="headers.length">
                        <br>
                        <h2 class="text-left"> Note : </h2>
                        <br>
                        <p class="text-left">{{ item.note }}</p>
                        <br>
                        <br>
                    </td>
                </template>

                <template v-slot:[`item.actions`]="{ item }">              
                    <v-btn style="background-color: transparent;" small class="mr-2" @click="editItem(item)">
                        <v-icon  color="green">edit</v-icon>
                    </v-btn>
                    <v-btn style="background-color: transparent;" small @click="deleteItem(item)">
                        <v-icon color="red">delete_forever</v-icon>
                    </v-btn>
                </template>
                
            </v-data-table>
        </v-card>
        <br>

        <v-card v-if="kepilih.length">
            <v-card-title>
                <h4>Delete Multiple</h4>
            </v-card-title>
            <v-list-item v-for="(item,i) in kepilih" :key="i">
                <div class="test">
                <v-list-item-content>
                    <v-list-item-title>•{{item.task}}</v-list-item-title>
                </v-list-item-content>
                </div>
            </v-list-item>
            <template>
                <v-row style="margin-left: 15px; "
                    align-md="start">
                    <v-btn small color="error" elevation="11" class="ma-2" @click="deleteAll">
                    Hapus Semua
                    </v-btn>
                </v-row>
            </template>
        </v-card>
        


        <v-dialog v-model="dialog" persistent max-width="600px">
            <v-card>
                <v-card-title>
                    <span class="headline">Form Todo</span>
                </v-card-title>
                <v-card-text>
                    <v-container>
                        <v-text-field
                            v-model="formTodo.task"
                            label="Task"
                            required>
                        </v-text-field>

                        <v-select
                            v-model="formTodo.priority"
                            :items="['Penting', 'Biasa', 'Tidak penting']"
                            label="Priority"
                            required>
                        </v-select>

                        <v-textarea
                            v-model="formTodo.note"
                            label="Note"
                            required>
                        </v-textarea>
                    </v-container>
                </v-card-text>

                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="blue darken-1" text @click="cancel">
                        Cancel
                    </v-btn>
                    <v-btn v-if="editCheck == true" color="blue darken-1" text @click="save">
                        Save
                    </v-btn>
                    <v-btn v-else color="blue darken-1" text @click="edit(formTodo)">
                        Save
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>

        <v-dialog v-model="deleteDialog" persistent max-width="400px">
            <v-card>
                <v-card-title > 
                    <span class="headline font-weight-bold">Yakin ingin menghapus?</span>
                </v-card-title>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="green darken-1" text @click="cancel">
                        Tidak
                    </v-btn>
                    <v-btn color="red darken-1" text @click="del">
                        Ya
                    </v-btn>                    
                </v-card-actions>
            </v-card>
        </v-dialog>

    </v-main>
</template>


<script>
    export default {
        name: "List",
        data() {
            return {
                expanded: [],
                singleExpand: false,
                search: null,
                dialog: false,
                kepilih:[],
                deleteDialog: false,
                detailDialog: false,
                itemTemp: null,
                editCheck: true,

                headers: [
                    { text: '', value: 'data-table-expand' },
                    {
                        text: "Task",
                        align: "start",
                        sortable: true,
                        value: "task",
                    },
                    
                    { text: "Priority", value: "priority" },                    
                    { text: "Actions", value: "actions" },
                    { text: "", value: "checks" },
                    
                ],

                todos: [
                    {
                        task: "bernafas",
                        priority: "Penting",
                        note: "huffttt",
                    },
                    {
                        task: "nongkrong",
                        priority: "Tidak penting",
                        note: "bersama tman2",
                    },
                    {
                        task: "masak",
                        priority: "Biasa",
                        note: "masak air 500ml",
                    },
                ],

                formTodo: {
                    task: null,
                    priority: null,
                    note: null,
                },

                
            };
        },

        methods: {
            save() {
                this.todos.push(this.formTodo);
                this.resetForm();
                this.dialog = false;
            },
            cancel() {
                this.resetForm();
                this.dialog = false;
                this.deleteDialog = false;
                this.detailDialog = false;
                this.editCheck = true;
                this.itemTemp = null;
            },
            resetForm() {
                this.formTodo = {
                    task: null,
                    priority: null,
                    note: null,
                };
            },
            editItem(item) {
                this.editCheck = false;
                this.dialog = true;
                this.formTodo = {
                    task : item.task,
                    priority : item.priority,
                    note : item.note,
                };
                this.itemTemp = item;
            },
            edit(formTodo) {
                this.itemTemp.task = formTodo.task;
                this.itemTemp.priority = formTodo.priority;
                this.itemTemp.note = formTodo.note;
                this.cancel();            
            },
            deleteItem(item) {
                this.deleteDialog = true;
                this.itemTemp = item;
            },
            del() {
                this.todos.splice(this.todos.indexOf(this.itemTemp), 1);
                this.dialogdel = false;
                this.cancel();
            },

            selected(item){
                if(this.kepilih.includes(item)){
                    this.kepilih.splice(this.kepilih.indexOf(item),1);
                    this.hapus=false
                }else{
                    this.hapus=true
                    this.kepilih.push(item);
                }
            },
            
            deleteAll(){
                var selected = this.kepilih.slice(0);

                for (var i = 0; i < selected.length; ++i) {
                    this.todos.splice( this.todos.indexOf(this.kepilih[i]), 1)
                }
                for(var i = 0; i < selected.length; ++i){
                    this.kepilih.splice(this.kepilih.indexOf(this.kepilih),1)
                }
            },

            urutkan: function(urut){
                function compare(pertama){
                    if(urut == "Penting")
                    {
                        if(pertama.priority == "Penting")
                            return -1;
                    }
                    else if(urut == "Tidak penting")
                    {
                        if(pertama.priority == "Tidak penting")
                            return -1;
                    }
                }
                return this.todos.sort(compare);
            }, 
        },
    };
</script>

<style scoped>
.test {
 display: grid;
 grid-template-columns: 200px 1fr;
 text-align: left;
}