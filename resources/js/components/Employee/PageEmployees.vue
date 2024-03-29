<template>
    <div class="employee">
        <h2 class="employee__title">Employees</h2>
        <div class="employee__options">
            <div class="search">
                <i class="fas fa-search search__icon"></i>
                <Input label="Search" type="text" class="search__input" placeholder="Search by Employee Name" @input="searchEmployee" v-model="search" />
            </div>
            <div class="actions">
                <button class="actions__add" @click="openModal($event, false)">
                    <i class="fas fa-plus"></i>
                </button>
                <button class="actions__edit" @click="openModal($event, true)" >
                    <i class="fas fa-edit"></i>
                </button>
                <button class="actions__delete" @click="deleteEmployee">
                    <i class="fas fa-trash"></i>
                </button>
            </div>
        </div>
        <div class="employee__container">
            <Card v-for="(employee, index) in this.employees" :key="index" :title="(`${employee.first_name} ${employee.middle_name[0]} ${employee.last_name}`)" :profile="employee.profile" :data="employee" @click.native="toggleSelect($event, index)" @dblclick.native="openDetails($event, index)"/>
        </div>
        <Modal v-if="this.formShow" ref="employeeModal" :showModal="formShow" @onClose="closeModal">
            <EmployeeForm :employee="this.employee" @addEmployee="loadEmployee" @editEmployee="updateEmployee" :edit="this.edit" />
        </Modal>
        <Modal ref="employeeDetails" :showModal="detailsShow" @onClose="closeModal">
            <EmployeeDetails :employee="this.employee" />
        </Modal>
    </div>
</template>
<script>
    import Modal from '../Common/Modal.vue'
    import Input from '../Common/Input.vue'
    import Card from '../Common/Card.vue'
    import EmployeeForm from './EmployeeForm.vue'
    import EmployeeDetails from './EmployeeDetails.vue'

    const dateFormat = (date) => {
        let format = [{year: 'numeric'}, {month: 'numeric'}, {day: 'numeric'}];
        return format.map(val => {
            return new Intl.DateTimeFormat('es', val).format(date)
        }).join("-");
    }

    const defaultValue = {
        id: 0,
        first_name: "",
        middle_name: "",
        last_name: "",
        phone: "",
        date_of_birth: "",
        gender: "",
        email: "",
        profile: null,
        role: "",
        department: "",
        salary: "0.00",
        salary_rate: "",
        hire_at: dateFormat(Date.now()),
        address: {
            address1: "",
            address2: "",
            city: "",
            state: "",
            country: "",
            zipcode: "",
        },
        user: {
            email: "",
            password: ""
        }
    };

    export default {
        components: {
            Card,
            Input,
            Modal,
            EmployeeForm,
            EmployeeDetails
        },
        data() {
            return {
                formShow: false,
                detailsShow: false,
                // Search Engine
                search: "",
                list: [],
                // Employee Values
                employees: [],
                employee: defaultValue,
                edit: false
            };
        },
        created() {
            axios.get(`${process.env.MIX_APP_URL}/api/employees`).then((response) => {
                //TODO: remove for loop to optimize the first instance
                for (let index = 0; index < response.data.length; index++) {
                    response.data[index] = {...response.data[index], selected: false}
                }
                this.employees = response.data;
            });
        },
        methods: {
            openModal(e, edit) {
                this.resetEmployee();
                this.edit = false;
                if(edit) {
                    this.employee = this.employees.find(val => {
                        return val.selected == true
                    });
                    if (!this.employee)
                        return
                    this.edit = true;
                }
                this.formShow = true;
            },
            openDetails(e, index) {
                this.employee = this.employees[index];
                this.detailsShow = true;
            },
            closeModal() {
                this.formShow = false;
                this.detailsShow = false;
                this.resetEmployee();
            },
            toggleSelect(e, index) {
                this.employees[index].selected = !this.employees[index].selected
            },
            loadEmployee(data) {
                data.selected = false;
                this.formShow = false;
                this.employees = [data, ...this.employees];
                this.$emit('add-notification', {
                    type: "success",
                    icon: "fa-user",
                    title: "Added",
                    sub: "The user has been added successfully"
                })
            },
            updateEmployee(data) {
                console.log(data);
                this.formShow = false;
                this.employee.profile = data.profile;
                this.$emit('add-notification', {
                    type: "success",
                    icon: "fa-user",
                    title: "Updated",
                    sub: "The user has been updated successfully"
                })
            },
            resetEmployee() {
                this.employee = defaultValue
            },
            searchEmployee(){
                if(this.list.length == 0) {
                    this.list = this.employees;
                }
                this.employees = this.list.filter(employee => {
                    return `${employee.first_name.toLowerCase()} ${employee.middle_name.toLowerCase()} ${employee.last_name.toLowerCase()}`.includes(this.search.toLowerCase());
                })
            },
            deleteEmployee() {
                const selectedEmployees = this.employees.filter(employee => {
                    return employee.selected
                });
                selectedEmployees.forEach(employee => {
                    axios.delete(`${process.env.MIX_APP_URL}/api/employees/${employee.id}`)
                    .then(res => {
                        let index = this.employees.indexOf(employee);
                        this.employees.splice(index, 1);
                    })
                })
            },
            addNotification() {
                this.$emit('add-notification', {
                    type: "info",
                    icon: "fa-user",
                    title: "Test",
                    sub: "Im just testing you know"
                })
            }
        },
    };
</script>
