<template>
    <div
        class="modal fade"
        :class="{ show: bModal }"
        id="modal-default"
        aria-modal="true"
        role="dialog"
    >
        <div class="modal-dialog modal-lg">
            <div class="modal-content">
                <div class="modal-header bg-success">
                    <h4 class="modal-title" v-text="tituloModal"></h4>
                    <button
                        type="button"
                        class="close"
                        data-dismiss="modal"
                        aria-label="Close"
                        @click="cierraModal"
                    >
                        <span aria-hidden="true">×</span>
                    </button>
                </div>
                <div class="modal-body">
                    <form>
                        <div class="row">
                            <div class="form-group col-md-12">
                                <label
                                    :class="{
                                        'text-danger': errors.nombre,
                                    }"
                                    >Nombre Oficina*</label
                                >
                                <el-input
                                    placeholder="Nombre"
                                    :class="{ 'is-invalid': errors.nombre }"
                                    v-model="oficina.nombre"
                                    clearable
                                >
                                </el-input>
                                <span
                                    class="error invalid-feedback"
                                    v-if="errors.nombre"
                                    v-text="errors.nombre[0]"
                                ></span>
                            </div>
                            <div class="form-group col-md-12">
                                <label
                                    :class="{
                                        'text-danger': errors.descripcion,
                                    }"
                                    >Descripción</label
                                >

                                <el-input
                                    type="textarea"
                                    placeholder="Descripción"
                                    :class="{
                                        'is-invalid': errors.descripcion,
                                    }"
                                    v-model="oficina.descripcion"
                                    clearable
                                    autosize
                                >
                                </el-input>
                                <span
                                    class="error invalid-feedback"
                                    v-if="errors.descripcion"
                                    v-text="errors.descripcion[0]"
                                ></span>
                            </div>
                            <div class="form-group col-md-12">
                                <label
                                    :class="{
                                        'text-danger': errors.dependencia_id,
                                    }"
                                    >Dependencia*</label
                                >

                                <el-select
                                    placeholder="Dependencia"
                                    class="w-100"
                                    :class="{
                                        'is-invalid': errors.dependencia_id,
                                    }"
                                    v-model="oficina.dependencia_id"
                                    clearable
                                >
                                    <el-option
                                        v-for="item in listDependencias"
                                        :key="item.id"
                                        :value="item.id"
                                        :label="item.nombre"
                                    ></el-option>
                                </el-select>
                                <span
                                    class="error invalid-feedback"
                                    v-if="errors.dependencia_id"
                                    v-text="errors.dependencia_id[0]"
                                ></span>
                            </div>
                        </div>
                    </form>
                </div>
                <div class="modal-footer justify-content-between">
                    <button
                        type="button"
                        class="btn btn-default"
                        data-dismiss="modal"
                        @click="cierraModal"
                    >
                        Cerrar
                    </button>
                    <el-button
                        type="danger"
                        class="bg-danger"
                        :loading="enviando"
                        @click="setRegistroModal()"
                        >{{ textoBoton }}</el-button
                    >
                </div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    props: {
        muestra_modal: {
            type: Boolean,
            default: false,
        },
        accion: {
            type: String,
            default: "nuevo",
        },
        oficina: {
            type: Object,
            default: {
                nombre: "",
                descripcion: "",
                dependencia_id: "",
            },
        },
    },
    watch: {
        muestra_modal: function (newVal, oldVal) {
            this.errors = [];
            if (newVal) {
                this.bModal = true;
            } else {
                this.bModal = false;
            }
        },
    },
    computed: {
        tituloModal() {
            if (this.accion == "nuevo") {
                return "AGREGAR OFICINA";
            } else {
                return "MODIFICAR REGISTRO";
            }
        },
        textoBoton() {
            if (this.accion == "nuevo") {
                return "Registrar";
            } else {
                return "Actualizar";
            }
        },
    },
    data() {
        return {
            user: JSON.parse(localStorage.getItem("user")),
            bModal: this.muestra_modal,
            enviando: false,
            listExpedido: [
                { value: "LP", label: "La Paz" },
                { value: "CB", label: "Cochabamba" },
                { value: "SC", label: "Santa Cruz" },
                { value: "CH", label: "Chuquisaca" },
                { value: "OR", label: "Oruro" },
                { value: "PT", label: "Potosi" },
                { value: "TJ", label: "Tarija" },
                { value: "PD", label: "Pando" },
                { value: "BN", label: "Beni" },
            ],
            listTipos: ["ADMINISTRADOR", "OPERADOR"],
            listDependencias: [],
            errors: [],
        };
    },
    mounted() {
        this.bModal = this.muestra_modal;
        this.getDependencias();
    },
    methods: {
        getDependencias() {
            axios.get(main_url + "/admin/dependencias").then((response) => {
                this.listDependencias = response.data.dependencias;
            });
        },
        setRegistroModal() {
            this.enviando = true;
            try {
                this.textoBtn = "Enviando...";
                let url = main_url + "/admin/oficinas";
                let config = {
                    headers: {
                        "Content-Type": "multipart/form-data",
                    },
                };
                let formdata = new FormData();
                formdata.append(
                    "nombre",
                    this.oficina.nombre ? this.oficina.nombre : ""
                );
                formdata.append(
                    "descripcion",
                    this.oficina.descripcion ? this.oficina.descripcion : ""
                );
                formdata.append(
                    "dependencia_id",
                    this.oficina.dependencia_id
                        ? this.oficina.dependencia_id
                        : ""
                );

                if (this.accion == "edit") {
                    url = main_url + "/admin/oficinas/" + this.oficina.id;
                    formdata.append("_method", "PUT");
                }
                axios
                    .post(url, formdata, config)
                    .then((res) => {
                        this.enviando = false;
                        Swal.fire({
                            icon: "success",
                            title: res.data.msj,
                            showConfirmButton: false,
                            timer: 1500,
                        });
                        this.limpiaOficina();
                        this.$emit("envioModal");
                        this.errors = [];
                        if (this.accion == "edit") {
                            this.textoBtn = "Actualizar";
                        } else {
                            this.textoBtn = "Registrar";
                        }
                    })
                    .catch((error) => {
                        this.enviando = false;
                        if (this.accion == "edit") {
                            this.textoBtn = "Actualizar";
                        } else {
                            this.textoBtn = "Registrar";
                        }
                        if (error.response) {
                            if (error.response.status === 422) {
                                this.errors = error.response.data.errors;
                            }
                            if (
                                error.response.status === 420 ||
                                error.response.status === 419 ||
                                error.response.status === 401
                            ) {
                                window.location = "/";
                            }
                            if (error.response.status === 500) {
                                Swal.fire({
                                    icon: "error",
                                    title: "Error",
                                    html: error.response.data.message,
                                    showConfirmButton: false,
                                    timer: 2000,
                                });
                            }
                        }
                    });
            } catch (e) {
                this.enviando = false;
                console.log(e);
            }
        },
        cargaImagen(e) {
            this.oficina.foto = e.target.files[0];
        },
        // Dialog/modal
        cierraModal() {
            this.bModal = false;
            this.$emit("close");
        },
        limpiaOficina() {
            this.errors = [];
            this.oficina.nombre = "";
            this.oficina.descripcion = "";
            this.oficina.dependencia_id = "";
        },
    },
};
</script>

<style></style>
