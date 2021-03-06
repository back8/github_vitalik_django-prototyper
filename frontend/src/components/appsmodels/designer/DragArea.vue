<template>
    <div :style="{width:size.width, height:size.height}">

        <relations 
            :models="models" 
            :selected="selected_models" 
            :height="size.height" 
            :width="size.width"
            @click.native="unselect_model" />
        
        
        <model v-for="item in models"
                @click.native="select_model(item, $event)"
                @dblclick.native="on_dblclick(item, $event)"
                @dragstart.native="dragstart"
                @dragend.native="dragend(item, $event)"
                draggable="true"
                class="model"
                :class="{selected:item.selected}"
                :key="item.key"
                :model="item.model"
                :app="item.app"
                :style="{left: item.model.ui_left+'px', top: item.model.ui_top+'px'}">
        </model>
    </div>

</template>

<script>

    import _ from 'lodash'
    import {store} from '../../../store'
    import Model from './Model'
    import Relations from "./Relations"

    export default {
        name: 'dragarea',

        data() {
            return {
                selected_models: [],
                drag_start_pos: null,
            }
        },

        computed: {
            models() {
                let result = []
                let n = 0
                _.each(store.project.apps, (app) => {
                    _.each(app.models, (m) => {
                        if (m.ui_top == undefined) {
                            this.$set(m, 'ui_top', n * 40)
                            this.$set(m, 'ui_left', n * 40)
                        }
                        let key = app.name + '.' + m.name
                        result.push({
                            app: app,
                            model: m,
                            selected: _.indexOf(this.selected_models, key) != -1,
                            key: key
                        })
                        n += 1
                    })
                })
                return result
            },
            size() {
                let w = _.maxBy(this.models, m => m.model.ui_left).model.ui_left + 200
                let h = _.maxBy(this.models, m => m.model.ui_top).model.ui_top + 200
                return {
                    width: w + 'px',
                    height: h + 'px',
                }
            }
        },
        methods: {
            select_model(item, event) {
                let modifier = event.shiftKey || event.altKey || event.ctrlKey
                if (!modifier)
                    this.selected_models = [item.key]
                else
                    this.selected_models.push(item.key)
            },
            unselect_model() {
                this.selected_models = []
            },
            on_dblclick(item, event) {
                let key = item.key.split('.')
                this.$router.push({name: 'model', params: {app: key[0], model:key[1]}})
            },
            dragstart(e) {
                e.dataTransfer.setData('Text', 'model') // firefox needs this
                this.drag_start_pos = [e.screenX, e.screenY]
            },
            dragend(item, e) {
                let model = item.model
                let offsetX = e.screenX - this.drag_start_pos[0]
                let offsetY = e.screenY - this.drag_start_pos[1]
                if (item.selected) {
                    _.each(this.models, (item) => {
                        if (item.selected)
                            this.move_model(item.model, offsetX, offsetY)
                    })
                } else {
                    this.move_model(model, offsetX, offsetY)
                }
                
                this.drag_start_pos = null
            },
            move_model(model, offsetX, offsetY) {
                model.ui_left = Math.round((model.ui_left + offsetX) / 20) * 20
                model.ui_top = Math.round((model.ui_top + offsetY) / 20) * 20
                if (model.ui_left < 0) model.ui_left = 0
                if (model.ui_top < 0) model.ui_top = 0
            }
        },
        components: {
            Model,
            Relations,
        }
    }

</script>


<style>

</style>

