<template>
    <div>
        <!-- Content Header (Page header) -->
        <section class="content-header">
            <h1>添加角色<small></small></h1>
            <ol class="breadcrumb">
                <li><a href="#"><i class="fa fa-dashboard"></i> Level</a></li>
                <li class="active">Here</li>
            </ol>
        </section>

        <!-- Main content -->
        <section class="content">
            <div class="row">
                <div class="col-xs-12">
                    <div class="box box-info">
                        <div class="box-header with-border">
                            <h3 class="box-title"> </h3>
                        </div>
                        <form style="max-width: 400px;margin: auto">
                            <div class="box-body">
                                <transition  enter-active-class="animated tada"
                                             leave-active-class="animated bounceOutRight">
                                    <div class="alert alert-success alert-dismissible" v-if="returnSuccess=='yes'">
                                        <button type="button" class="close" @click="returnSuccess=no"
                                                aria-hidden="true">×
                                        </button>
                                        <h4><i class="icon fa fa-check"></i> 提示!</h4>
                                        {{returnMSG}}
                                    </div>
                                </transition>

                                <div class="form-group"  v-bind:class="{'has-error': errors.has('name') }">
                                    <transition enter-active-class="animated shake"
                                                leave-active-class="xx">
                                        <label class="control-label" v-show="errors.has('name')">
                                            <i class="fa fa-times-circle-o"></i> {{ errors.first('name') }}
                                        </label>
                                    </transition>
                                    <div class="input-group">
                                        <span class="input-group-addon"><i class="fa fa-font"></i></span>
                                        <input type="text" class="form-control" v-model="name" name="name" v-validate="{ rules: { required: true,alpha_dash:true,min: 4,max:16 } }" data-vv-as="角色名" placeholder="角色名">
                                    </div>
                                </div>
                                <div class="form-group"  v-bind:class="{ 'has-error': errors.has('displayName') }">
                                    <transition enter-active-class="animated shake"
                                                leave-active-class="xx">
                                        <label class="control-label" v-show="errors.has('displayName')">
                                            <i class="fa fa-times-circle-o"></i> {{ errors.first('displayName') }}
                                        </label>
                                    </transition>
                                    <div class="input-group">
                                        <span class="input-group-addon"><i class="fa fa-text-height"></i></span>
                                        <input type="text" class="form-control" v-model="displayName" name="displayName" v-validate="{ rules: { required: true,min: 2,max:16 } }" data-vv-as="角色显示名称" placeholder="角色显示名称">
                                    </div>
                                </div>
                                <div class="form-group"  v-bind:class="{ 'has-error': errors.has('description') }">
                                    <transition enter-active-class="animated shake"
                                                leave-active-class="xx">
                                        <label class="control-label" v-show="errors.has('description')">
                                            <i class="fa fa-times-circle-o"></i> {{ errors.first('description') }}
                                        </label>
                                    </transition>
                                    <div class="input-group">
                                        <span class="input-group-addon"><i class="fa fa-align-left"></i></span>
                                        <textarea type="text" class="form-control" v-model="description" name="description" v-validate="{ rules: {max:40 } }" data-vv-as="描述" placeholder="描述"></textarea>
                                    </div>
                                </div>
                                <h3>附加权限</h3>
                                <div class="form-group">
                                    <div class="row">
                                        <div class="col-xs-6" v-for="item in allPermissions">
                                            <label>
                                                <input type="checkbox" class="minimal" :value="item.id" :id="item.id" v-model="permissionsChecked">
                                                {{item.display_name}}
                                            </label>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <!-- /.box-body -->
                            <div class="box-footer">
                                <button type="button" class="btn btn-default" @click="resetting">重置</button>
                                <button type="button" class="btn btn-info pull-right" @click="add">添加</button>
                            </div>
                            <!-- /.box-footer -->
                        </form>
                    </div>
                </div>
                <!-- /.col -->
            </div>
            <!-- /.row -->
        </section>
        <!-- /.content -->
    </div>
</template>
<script>
    export default {
        data:function () {
            return {
                name: '',
                displayName: '',
                description: '',
                returnSuccess: '',
                returnMSG: '',
                allPermissions: [],
                permissionsChecked:[]
            }
        },
        computed: {
        },
        created:function (){
            var that = this;
            this.$nextTick(function () {
                this.loadPermission();
            });
        },

        watch: {
            '$route': function () {

            }
        },
        methods: {
            add:function () {
                this.$validator.validateAll().then(success => {
                    if (! success) {
                        // handle error
                        return;
                    }
                    var param = {
                        name        : this.name,
                        displayName : this.displayName,
                        description : this.description,
                        permissionsChecked : this.permissionsChecked
                    };
                    this.$http.post(this.$store.state.siteUrl+'/addRole', param).then(function(response){
                        var data = response.data;
                        var that = this;
                        if(data.code==20000){
                            this.returnSuccess = 'yes';
                            this.returnMSG = data.msg;
                            this.resetting();
                            setTimeout(function () {
                                that.errors.clear();
                            },10);
                            $('input').icheck('unchecked');

                        }else{
                            this.returnSuccess = 'no';
                            this.returnMSG = data.msg;
                            for(var key in data.msg){
                                this.errors.errors.unshift({field:key,msg:data.msg[key][0],scope:"__global__"})
                            }
                        }
                    }).catch(function(response) {
                        this.returnSuccess = 'no';
                        this.returnMSG = '系统出错，请稍后再试！';
                    });
                });
            },
            resetting:function () {
                this.name = '';
                this.displayName = '';
                this.description = '';
                $('input').icheck('unchecked');
            },
            loadPermission:function () {
                    this.$http.get(this.$store.state.siteUrl+'/allPermissions').then(function(response){
                        var data = response.data;
                        var that = this;
                        if(data.code==20000){
                            that.allPermissions = data.data;
                            setTimeout(function () {
                                $('input').icheck({
                                    checkboxClass: 'icheckbox_square-blue',
                                    radioClass: 'iradio_square-blue',
                                    increaseArea: '20%' // optional
                                });
                            },1);
                        }
                    }).catch(function(response) {
                        this.returnSuccess = 'no';
                        this.returnMSG = '系统出错，请稍后再试！';
                    });
            }


        },
        components: {

        }
    }
</script>