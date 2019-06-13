<template>
 <div class="book">
    
    <div class="searchContainer">
        <button @click="search">Поиск</button>
        <input type="text" v-model="searchValue">
    </div>
        
    
    
    <table border="1">
       <thead>
           <tr>
               <th>ID</th>
               <th>Phone</th>
               <th>Name</th>
               <th>E-mail</th>
               <th>Редактировать</th>
               <th>Удалить</th>
           </tr>
       </thead>
        <tbody>
        <tr v-for="user in users">
            <td>{{user.id}}</td>
            <td>{{user.phone}}</td>
            <td>{{user.name}}</td>
            <td>{{user.email}}</td>
            <td><button @click="showEditUserModal({...user})">Ред.</button></td>
            <td class="delete" @click="showDeleteUserModal(user.id)"><img src="../assets/delete.png" alt=""></td>
        </tr>
        </tbody>
    </table>
    
    <button type="button" @click="showAddUserModal()">Добавить пользователя</button>
<!--    <button type="button" @click="getUsers">Обновить лист</button>    -->
    
 </div>
</template>

<script>
    const axios = require('axios');
    var link = 'https://phone-book-decentury.herokuapp.com/api';
    var proxyUrl = 'https://cors-anywhere.herokuapp.com/';

    export default {
        name: 'Book',
        data() {
            return {
                users: [],
                backupUsers: [],
                searchValue: ''
            }
        },
        mounted: function() {
            this.getUsers();
        },
        created: function() {},
        methods: {

            showAddUserModal() {
                this.$modal.show({
                    template: `
                            <div>
                               <center>
                                   <h1>Добавить пользователя</h1>
                               </center>
                               <center>
                                   <input type="text" v-model="userCreateData.name" placeholder="Имя">
                               </center>
                               <br/>
                               <center>
                                   <input type="text" v-model="userCreateData.phone" placeholder="Телефон">
                               </center>
                               <br/>
                               <center>
                                   <input type="text" v-model="userCreateData.email" placeholder="E-mail">
                               </center>
                               <br/>
                               <center>
                                   <button @click="addUser(userCreateData)">Добавить</button>
                               </center>
                            </div>
                            `,
                    props: ['userCreateData', 'addUser']
                }, {
                    userCreateData: {
                        name: '',
                        phone: '',
                        email: ''
                    },
                    addUser: this.addUser
                }, {
                    name: 'addUserModal'
                })
            },
            showEditUserModal(user) {

                this.$modal.show({
                    template: `
                            <div>
                               <center>
                                   <h1>Редактировать пользователя {{user.id}}</h1>
                               </center>
                               <br/>
                               <center>
                                   <input id="userName" v-model="user.name" type="text" placeholder="Имя">
                               </center>
                               <br/>
                                <center>
                                   <input id="userPhone" v-model="user.phone" type="text" placeholder="Телефон">
                               </center>
                               <br/>
                                <center>
                                   <input id="userMail" v-model="user.email" type="text" placeholder="E-mail">
                               </center>
                               <br/>
                               <center>
                                   <button @click="editUser(user)">Редактировать</button>
                                   <button @click="$emit('close')">Отменить</button>
                               </center>
                            </div>
                            `,
                    props: ['user', 'editUser']
                }, {
                    user,
                    editUser: this.editUser
                }, {
                    name: 'editUserModal'
                })
            },
            showDeleteUserModal(id) {

                this.$modal.show({
                    template: `
                            <div>
                                <center>
                                   <h1>Вы действительно хотите удалить пользователя {{id}}?</h1>
                                </center>
                                <br/>
                                <center>
                                   <button @click="deleteUser(id)">Ok</button>
                                   <button @click="$emit('close')">Cancel</button>
                                </center>
                            </div>
                            `,
                    props: ['id', 'deleteUser']
                }, {
                    id,
                    deleteUser: this.deleteUser
                }, {
                    name: 'deleteUserModal'
                })
            },
            showErrorModal(errorMessage) {

                this.$modal.show({
                    template: `
                            <div>
                                <center>
                                   <h1>Error: {{errorMessage}}</h1>
                                </center>
                                <br/>
                                <center>
                                   <button @click="$emit('close')">Ok</button>
                                </center>
                            </div>
                            `,
                    props: ['errorMessage', 'deleteUser']
                }, {
                    errorMessage,
                    deleteUser: this.deleteUser
                }, {
                    name: 'errorModal'
                })
            },

            getUsers() {
                axios.get(`${link}/pb`).then(response => {
                    const temp = [];

                    for (var key in response.data) {
                        if (response.data[key] != null) {
                            var user = {
                                id: response.data[key].id,
                                phone: response.data[key].phone,
                                name: response.data[key].name,
                                email: response.data[key].email
                            };

                            temp.push(user);
                        }
                    }
                    this.users = temp;
                    this.backupUsers = [...this.users]
                })
            },
            editUser(user) {
                axios.put(`${proxyUrl}${link}/pb`, {
                        data: user
                    })
                    .then(this.$modal.hide('editUserModal'))
                    .then(this.getUsers)
                    .catch(response => this.showErrorModal(response.response.data.message))
            },
            addUser(userCreateData) {
                axios({
                        method: 'post',
                        url: `${link}/pb/`,
                        data: {
                            data: userCreateData
                        }
                    })
                    .then((response) => {
                    this.$modal.hide('addUserModal'); 
                    this.getUsers()}, 
                          (response) => this.showErrorModal(response.response.data.message))
                    
            },
            deleteUser(id) {
                axios.delete(`${proxyUrl}${link}/pb/${id}`)
                    .then(this.$modal.hide('deleteUserModal'))
                    .then(this.getUsers)
                    .catch(error => console.log(error))
            },
            search() {
                if (this.users.length != this.backupUsers.length) {
                    this.users = [...this.backupUsers];
                }
                if (this.searchValue.length) {
                    this.users = this.users.filter(user => (user.name.toLowerCase().includes(this.searchValue.toLowerCase()) || user.phone.toLowerCase().includes(this.searchValue.toLowerCase()) || user.email.toLowerCase().includes(this.searchValue.toLowerCase())))
                }
            }
        }
    }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
    table {
        width: 100%;
        max-width: 1100px;
        margin: 40px auto;
    }

    .delete {
        padding: 2px;
        cursor: pointer;
        font-size: 0;
    }

    .delete img {
        max-width: 20px;
        opacity: 0.7;
    }

    .delete img:hover {
        opacity: 1;
    }

</style>