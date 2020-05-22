<!-- This is a Vue.js single file component. -->
<!-- Check the Vue.js doc here :  -->
<!-- https://vuejs.org/v2/guide/ -->

<!-- This is your HTML -->
<template>
    <div>
        <div class="section-container">
            <!-- wwManager:start -->
            <wwSectionEditMenu :sectionCtrl="sectionCtrl"></wwSectionEditMenu>
            <!-- wwManager:end -->

            <!-- This is the background of the section -->
            <wwObject class="background"
                      :ww-object="section.data.background"
                      ww-category="background"></wwObject>

            <div class="content">
                <!-- This is a content block, a list of objects, it will add the little blue "plus" around the objects in the editor -->
                <ul class="left-content">
                    <li class="feature-wrapper"
                        v-for="(feature, index) in section.data.features"
                        :key="feature.uniqueId">
                        <!-- wwManager:start -->
                        <wwContextMenu v-if="editMode"
                                       class="ww-orange-button"
                                       tag="div"
                                       :options="elemOptions"
                                       @remove="removeFeature(feature)"
                                       @addBefore="addFeatureBefore(index)"
                                       @addAfter="addFeatureAfter(index)">
                            <wwOrangeButton></wwOrangeButton>
                        </wwContextMenu>
                        <!-- wwManager:end -->
                        <div class="feature-logo">
                            <wwObject :ww-object="feature.logo"></wwObject>
                        </div>
                        <div class="feature-title"
                             :class="{ selected: isFeatureSelected(feature,selectedFeature) }"
                             @click="toggleFeature(feature)">
                            <svg width="12" height="8" viewBox="0 0 12 8" xmlns="http://www.w3.org/2000/svg"
                                 class="toggle-icon">
                                <path d="M1.41 0.589966L6 5.16997L10.59 0.589966L12 1.99997L6 7.99997L0 1.99997L1.41 0.589966Z"/>
                            </svg>
                            <wwObject tag="span"
                                      :ww-object="feature.title"
                            ></wwObject>
                        </div>
                        <wwLayoutColumn tag="div"
                                        ww-default="ww-text"
                                        :ww-list="feature.contentList"
                                        @ww-add="add(feature.contentList, $event)"
                                        @ww-remove="remove(feature.contentList, $event)">
                            <wwObject tag="div"
                                      ww-default="ww-text"
                                      class="feature-content"
                                      :class="{ selected: isFeatureSelected(feature,selectedFeature) }"
                                      v-for="item in feature.contentList"
                                      :key="item.uniqueId"
                                      :ww-object="item"
                            ></wwObject>
                        </wwLayoutColumn>
                    </li>

                </ul>
                <div class="right-wrapper">
                    <wwObject class="background"
                              :ww-object="section.data.rightBackground"
                              ww-category="background">

                    </wwObject>
                    <transition name="fade">
                        <div class="feature-display-wrapper">
                            <wwObject class="feature-display-content"
                                      v-for="(feature, index) in section.data.features" :key="index"
                                      :class="{active:isFeatureSelected(feature,selectedFeature)}"
                                      :ww-object="feature.display"></wwObject>
                        </div>
                    </transition>
                </div>
            </div>
        </div>
    </div>
</template>

<!-- This is your Javascript -->
<!-- ✨ Here comes the magic ✨ -->
<script>

    const wwo = window.wwLib.wwObject;
    const wwu = window.wwLib.wwUtils;

    export default {
        name: "__COMPONENT_NAME__",
        props: {
            sectionCtrl: Object
        },
        data: () => ({
            selectedFeature: {},
            elemOptions: {

                items: [
                    {
                        text: {
                            en: 'Before',
                            fr: 'Avant'
                        },
                        icon: 'wwi wwi-add',
                        action: 'addBefore'
                    },
                    {
                        text: {
                            en: 'After',
                            fr: 'Apres'
                        },
                        icon: 'wwi wwi-add',
                        action: 'addAfter'
                    },
                    {
                        text: {
                            en: 'Delete',
                            fr: 'Supprimer'
                        },
                        icon: 'wwi wwi-delete',
                        action: 'remove'
                    }
                ]
            }
        }),
        computed: {
            section() {
                return this.sectionCtrl.get();
            },

            editMode() {
                return this.sectionCtrl.getEditMode() === 'CONTENT'
            },
        },
        created() {
            this.init()
        },
        mounted() {
            if (this.section.data.features.length > 0) {
                this.selectedFeature = this.section.data.features[0];
            }
        },
        methods: {
            init() {
                let needUpdate = false
                this.section.data = this.section.data || {};

                if (!this.section.data.background) {
                    this.section.data.background = wwo.getDefault({
                        type: 'ww-color'
                    });
                    needUpdate = true
                }

                if (!this.section.data.rightBackground) {
                    this.section.data.rightBackground = wwo.getDefault({
                        type: 'ww-image'
                    });
                    needUpdate = true
                }

                if (!this.section.data.features) {
                    this.section.data.features = [this.createFeature()]
                    needUpdate = true
                }
                needUpdate && this.update();
            },

            getNewFeature: () => ({
                uniqueId: wwu.getUniqueId(),
                logo: wwo.getDefault({
                    type: 'ww-row'
                }),
                title: wwo.getDefault({type: 'ww-text'}),
                contentList: [],
                display: wwo.getDefault({
                    type: 'ww-image'
                })
            }),

            createFeature() {
                let feature = {}
                if (this.section.data.features.length > 0) {
                    feature = JSON.parse(JSON.stringify(this.section.data.features[0]))
                    wwu.changeUniqueIds(feature)
                    feature.uniqueId = wwu.getUniqueId()
                    return feature
                }
                return this.getNewFeature()
            },

            isFeatureSelected(feature, activeFeature) {
                return feature.uniqueId === activeFeature.uniqueId
            },
            toggleFeature(feature) {
                this.selectedFeature = feature
            },
            addFeatureBefore(index) {
                this.addFeatureAt(index === 0 ? 0 : index - 1)
                this.update()
            },
            addFeatureAfter(index) {
                this.addFeatureAt(index + 1)
                this.update()
            },
            addFeatureAt(index) {
                const {features} = this.section.data
                const head = features.slice(0, index)
                const tail = index === 0 ? features : features.slice(index + 1)
                this.section.data.features = [...head, this.createFeature(), ...tail]
            },
            removeFeature(feature) {
                const {features} = this.section.data;
                if (features.length === 1) return
                this.section.data.features = features.filter(aFeature => aFeature !== feature)
                this.update()
            },
            update() {
                this.sectionCtrl.update(this.section);
            },

            /* wwManager:start */
            add(list, options) {
                list.splice(options.index, 0, options.wwObject);
                this.update()
            },
            remove(list, {index}) {
                list.splice(index, 1);
                this.update()
            }
            /* wwManager:end */
        }
    };
</script>

<style lang="scss" scoped>

    .section-container {
        padding:24px;
        margin: auto;
        color: var(--color-true-black);
        @media (min-width: 769px) {
            padding: 64px;
        }
    }

    .background {
        position: absolute;
        top: 0;
        left: 0;
        height: 100%;
        width: 100%;
    }

    .content {
        position: relative;
        display: flex;
        flex-direction: row;
        align-items: flex-start;
    }

    .left-content {
        position: relative;
        padding: 0;
        width: 100%;
        list-style-type: none;
        @media (min-width: 1025px) {
            width: 50%;
        }
    }

    .feature {
        &-wrapper {
            position: relative;
            margin-bottom: 24px;
            @media (min-width: 1025px) {
                margin-bottom: 48px;
            }
        }

        &-logo {
            margin-left: 32px;
            margin-bottom: 16px;
        }

        &-title {
            display: flex;
            flex-direction: row;
            align-content: center;
            align-items: center;
            width: fit-content;
            margin-bottom: 12px;
            color: black;
            font-size: 18px;
            line-height: 1.2;
            pointer-events: all;

            @media (min-width: 1025px) {
                font-size: 24px;
                line-height: 28px;
            }

            &.selected {
                color: var(--color-red);

                .toggle-icon {
                    transform: rotateZ(-90deg);
                    transition: all 250ms;

                    path {
                        fill: var(--color-red);
                    }
                }
            }
        }
    }

    .feature-content {
        max-height: 0;
        margin-left: 32px;
        overflow: hidden;
        font-size: 16px;
        line-height: 1.2;
        transition: max-height 250ms;

        @media (min-width: 1025px) {
            font-size: 18px;
            line-height: 21px;
        }

        &.selected {
            max-height: 100px;
            transition: max-height 250ms;
        }
    }

    .right-wrapper {
        display: none;
        @media (min-width: 1025px) {
            display: block;
            width: 50%;
            position: relative;
        }
    }

    .feature-display {
        &-wrapper {
            position: relative;
            width: 496px;
            height: 640px;
            overflow: hidden;
            margin: auto;
        }

        &-content {
            position: absolute;
            top: 0;
            right: 0;
            width: 100%;
            opacity: 0;

            &.active {
                opacity: 1;
                z-index: 2;
            }
        }
    }


    .toggle-icon {
        display: inline-block;
        width: 16px;
        height: 16px;
        margin-right: 16px;

        path {
            fill: var(--color-true-black);
        }
    }

</style>
