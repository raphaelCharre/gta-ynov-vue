<template>
    <div class="">
        <nav-bar></nav-bar>
        <b-tabs card>
            <b-tab title="Comptes" class="account-tab" active>
                <user-management>
                </user-management>
            </b-tab>


            <b-tab title="Equipes" >
                <team-management 
                    :teams="teams"
                    @onCreateTeam="onCreateTeam"
                    @onRemoveTeam="onRemoveTeam">
                </team-management>
            </b-tab>


            <b-tab title="Contrats" >
                <div class="row">
                    <div class="col-md-3">
                        <div class="form-group">
                            <label for="occupation-selector">Utilisateur: </label>
                            <select @change="onSelectUser" class="form-control" name="user-contract-selector" v-model="selected_user_contract">
                                <option :value="null">-- Selectionner un utilisateur --</option>
                                <option v-for="user in users" :key="user.id" :value="user">{{user.fname}} {{user.lname}}</option>
                            </select>
                        </div>

                        <div v-if="selected_user_contract">
                            <label for="occupation-selector">Contrat: </label>
                            <select class="form-control" name="contract-selector" v-model="selected_contract">
                                <option :value="null">-- Selectionner un contrat --</option>
                                <option v-for="contract in contracts" :key="contract.id" :value="contract">{{contract.motif}}</option>
                            </select>
                            <p class="text-center">
                                <a v-b-popover.hover="'Creer un contrat'" 
                                    style="font-size: 24px;" 
                                    @click="createContract()" 
                                    class="fas fa-plus-circle btn text-success"></a>
                            </p>
                        </div>                        
                    </div>

                    <div class="col-md-3" v-if="selected_contract">
                        <div class="form-group">
                            <label for="contract-motif">Motif: </label>
                            <input type="text" class="form-control" id="contract-motif" v-model="selected_contract.motif" required/>
                        </div>

                        <div class="form-group">
                            <label for="contract-dstart">Date de début: </label>
                            <date-picker :config="date_picker_options" class="form-control" name="contract-dstart" v-model="selected_contract.dateStart" required></date-picker>
                        </div>

                        <div class="form-group">
                            <label for="contract-dend">Date de fin: </label>
                            <date-picker :config="date_picker_options" class="form-control" name="contract-dend" v-model="selected_contract.dateEnd" required></date-picker>
                        </div>

                        <div class="form-group">
                            <label for="contract-hours-per-week">Heure hebdomadaire: </label>
                            <input type="number" class="form-control" id="contract-hours-per-week" v-model="selected_contract.hours_per_week" required/>
                        </div>
                    </div>

                    <div v-if="selected_contract" class="col-md-6">
                        <div class="form-group">
                            <b-btn @click="modifyContract" variant="success">Valider</b-btn>
                        </div>

                        <div v-for="day in 7" :key="day">
                            <b-card
                                :header="$moment().day(day).format('dddd')">
                                <div v-for="hour in 4" :key="hour">
                                        <div class="form-group">
                                        <input 
                                            type="time" 
                                            class="form-control" 
                                            v-model="selected_contract.hours[day-1][hour-1]" required/>
                                    </div>
                                </div>
                            </b-card>
                            <hr>
                        </div>
                        <div class="form-group">
                            <b-btn @click="modifyContract" variant="success">Valider</b-btn>
                        </div>
                    </div>
                </div>
            </b-tab>


            <b-tab title="Paramètrage" >
                <p class="display-4">Occupations: </p>
                <div class="col-md-4">
                    <div class="form-group">
                        <label for="occupation-selector">Occupation: </label>
                        <select class="form-control" name="occupation-selector" v-model="selected_occupation">
                            <option :value="null">-- Selectionner une occupation --</option>
                            <option v-for="occupation in occupations" :key="occupation.id" :value="occupation">{{occupation.name}}</option>
                        </select>
                        <p class="text-center">
                            <a v-b-popover.hover="'Creer une occupation'" 
                                style="font-size: 24px;" 
                                @click="createOccupation()" 
                                class="fas fa-plus-circle btn text-success"></a>
                        </p>
                    </div>

                    <div v-if="selected_occupation">
                        <div class="form-group">
                            <label for="admin-occupation-name">Nom: </label>
                            <input type="text" class="form-control" id="admin-occupation-name" v-model="selected_occupation.name" required/>
                        </div>

                        <div class="form-group">
                            <label for="admin-occupation-effective-hours">Heures comptées: </label>
                            <input type="number" class="form-control" id="admin-occupation-effective-hours" v-model="selected_occupation.effective_hours_mult" required/>
                        </div>
                        <b-btn @click="modifyOccupation" variant="success">Valider</b-btn>
                    </div>                          
                </div>
                <hr>
            </b-tab>

            <b-tab title="Logs">
                <div class="form-group">
                    <b-btn variant="info" @click="refreshLogs">Rafraichir</b-btn>
                </div>
                <table class="table table-stiped table-responsive-sm">
                    <thead>
                        <th>Utilisateur</th>
                        <th>Date</th>
                        <th>Action</th>
                        <th>Type</th>
                        <th>Details</th>
                    </thead>
                    <tbody>
                        <tr v-for="log in logs" :key="log.id">
                            <td>{{log.mail}}</td>
                            <td>{{$moment(log.date).format('DD/MM/YYYY HH:mm')}}</td>
                            <td>{{log.action}}</td>
                            <td>{{log.type}}</td>
                            <td>{{log.details}}</td>
                        </tr>
                    </tbody>
                </table>
            </b-tab>
        </b-tabs>
    </div>
    
</template>

<script>
import Occupation from '../entities/occupation'
import Contract from '../entities/contract'

import NavBar from '../layout/NavBar.vue'
import TeamManagement from '../layout/TeamManagement.vue'
import UserManagement from '../layout/UserManagement.vue'
import DatePicker from 'vue-bootstrap-datetimepicker'

import TeamService from '../services/team.service';
import LogService from '../services/log.service';
import ContractService from '../services/contract.service';
import OccupationService from '../services/occupation.service';
import UserService from '../services/user.service';
import contract from '../entities/contract';
export default {
    components:{
        NavBar, TeamManagement, UserManagement, DatePicker
    },
    data(){
        return{
            teams: [],

            users: [],
            contracts: [],
            logs:[],

            selected_user_contract: null,
            selected_contract: null,

            occupations: [],
            selected_occupation: null
        }
    },
    mounted(){
        TeamService.getAll(this.setTeams);
        UserService.getAll(this.setUsers);
        LogService.getAll(this.setLogs);
        OccupationService.getAll(this.setOccupations);
    },
    methods:{
        setLogs(logs){
            this.logs = logs.reverse();
        },
        refreshLogs(){
            LogService.getAll(this.setLogs);
        },

        setUsers(users){
            this.users = users;
        },
        onSelectUser(){
            ContractService.getContractsForUser(this.setContracts, this.selected_user_contract);
            this.selected_contract = null;
        },

        setContracts(contracts){
            this.contracts = contracts.map(contract=>{
                contract.dateStart = this.$moment(contract.dateStart);
                contract.dateEnd = this.$moment(contract.dateEnd);
                return contract;
            });
        },
        createContract(){
            var contract = new Contract(null, this.selected_user_contract.id, new Date(), new Date(), 'Nouveau contrat', null)
            ContractService.create(this.amIAuthentified(), this.onCreateContract, contract)
        },
        onCreateContract(contract){
            this.selected_contract = contract;
            ContractService.getContractsForUser(this.setContracts, this.selected_user_contract);
        },
        modifyContract(){
            ContractService.modify(this.amIAuthentified(), this.onModifyContract, this.selected_contract);
        },
        onModifyContract(contract){
            
        },

        setOccupations(occupations){
            this.occupations = occupations
        },
        createOccupation(){
            var occupation = new Occupation(null, 'Nouvelle occupation', 0);
            OccupationService.create(this.amIAuthentified(), this.onCreateOccupation, occupation);
        },
        onCreateOccupation(occupation){
            this.occupations.push(occupation);
            this.selected_occupation = occupation
        },
        modifyOccupation(){
            OccupationService.modify(this.amIAuthentified(), this.onModifyOccupation, this.selected_occupation);
        },
        onModifyOccupation(){

        },

        setTeams(teams){
            this.teams = teams;
        },
        onCreateTeam(team){
            this.teams.push(team);
        },
        onRemoveTeam(){
            TeamService.getAll(this.setTeams);
        }
    }
    
}
</script>

<style>
</style>
