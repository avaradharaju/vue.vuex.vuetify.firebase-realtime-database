<template >
    <div v-if="!this.$store.state.loadingIndicator">
        <v-toolbar color="blue" dark @click="formVisiblity=formVisiblity?0:1">
            <v-toolbar-title>{{entityConfig.title}}</v-toolbar-title><v-spacer></v-spacer><v-btn icon><v-icon >{{formVisiblity?'expand_less':'expand_more'}}</v-icon></v-btn>
        </v-toolbar>
        <v-expansion-panel v-model="formVisiblity">
            <v-expansion-panel-content >
            <v-card >
                <v-card-text>
                <v-form ref="profileForm">
                    <div v-for="(formField, index) in entityConfig.formFields" :key="index">
                    <text-field v-if="formField.fieldType == 'text'" :fieldName="formField.fieldName" :fieldLabel="formField.fieldLabel" :rules="formField.rules" :collectionId="entityConfig.collectionId" :docId="entityConfig.docId" :disabled="formField.disabled"></text-field>
                    <date-field v-if="formField.fieldType == 'date'" :fieldName="formField.fieldName" :fieldLabel="formField.fieldLabel" :rules="formField.rules" :collectionId="entityConfig.collectionId" :docId="entityConfig.docId"></date-field>
                    <checkbox-field v-if="formField.fieldType == 'checkbox'" :fieldName="formField.fieldName" :fieldLabel="formField.fieldLabel" :collectionId="entityConfig.collectionId" :docId="entityConfig.docId"></checkbox-field>
                    <select-field v-if="formField.fieldType == 'select'" :items="formField.fieldItems" :fieldName="formField.fieldName" :fieldLabel="formField.fieldLabel" :collectionId="entityConfig.collectionId" :docId="entityConfig.docId"></select-field>
                    </div>
                    <div class="text-xs-right">
                    <v-btn color="error" @click="confirmDialogVisibility=true">Delete<v-icon right>delete</v-icon></v-btn>
                    </div>
                </v-form>
                <template v-if="confirmDialogVisibility">
                <dialog-confirm confirmType="error" :confirmVisibilty="confirmDialogVisibility" @confirmAccept="fDelete" @confirmCancel="confirmDialogVisibility=false">
                    <template slot="title">Confirm Delete</template>
                    <template slot="text">{{entityConfig.onDelete.confirmMessage}}</template>
                    <template slot="confirmButton">Confirm Delete</template>
                </dialog-confirm> 
                </template>
                </v-card-text>
            </v-card> 
            </v-expansion-panel-content>
        </v-expansion-panel>
    </div>
</template>


<script>
import DialogConfirm from '@/components/shared/DialogConfirm';
import FormTextField from '@/components/shared/FormFields/Text';
import FormDateField from '@/components/shared/FormFields/Date';
import FormCheckboxField from '@/components/shared/FormFields/Checkbox';
import FormSelectField from '@/components/shared/FormFields/Select';

export default {
  name: 'EntityForm',
  props:['entityConfig'],
  components:{
    'dialog-confirm':DialogConfirm,
    'text-field':FormTextField,
    'date-field':FormDateField,
    'checkbox-field':FormCheckboxField,
    'select-field':FormSelectField,
  },
  data() {
    return {
      confirmDialogVisibility:false,
      formVisiblity:0,
      currentRoute:"",
    };
  },
  watch:{
    // When creating a new entity, the same component (TemplateLoader) is re-used by Vue and therefore the mounted/created functions are not re-run
    // The new entity is loaded when the entityConfig object updates and therefore entityConfig.entityUrl will be updated to match the new url of the newly created entity
    'entityConfig.entityUrl':function(newVal){
      if(this.$route.path == newVal){
        this.$store.commit('setLoadingIndicator', false);
        this.$nextTick(function () {
          // Need to wait until the next 'tick' before calling the validation
          // https://vuejs.org/v2/guide/reactivity.html#Async-Update-Queue
          this.$refs.profileForm.validate();
        })
      }
    }
  },
  methods:{
    fDelete(){
      this.confirmDialogVisibility = false;
      //TODO: need to handle failure - ie loss of connection to server
      this.$store.dispatch('fdeleteEntity',{collectionId:this.entityConfig.collectionId,docId:this.entityConfig.docId,route:{to:this.entityConfig.parentEntityUrl}})
    },
  },
  created(){
    if(this.entityConfig.entityUrl == this.$route.path){
      this.$store.commit('setLoadingIndicator', false);
      console.log('hide loader $route.path');
    }

    //console.log('EntityForm.vue created function. Props: ' + JSON.stringify(this.$options.propsData));
  },
  mounted(){
    //console.log('EntityForm.vue mounted function. Props: ' + JSON.stringify(this.$options.propsData));
    console.log('EntityForm.vue created function.');
    this.$refs.profileForm.validate();
  },
}
</script>

<style>
</style>