<template>
  <div class="page" :class="{ 'page--scroll' : scrollPage }">
    <div class="page__inner">
      <div class="wrapper">
        <div class="d-flex">
          <h1>Dropdown</h1>
          <button @click="scrollPage = !scrollPage">toggle scrollPage</button>
        </div>

        <div v-for="(item, index) in items" :key="index">
          <h2>{{ item.title }}</h2>
          <div class="d-flex margin-b-20">
            <div v-for="(item, index) in item.items" :key="index" class="margin-h-20">
              <Dropdown :placement="item.placement" trigger-class="d-inline-flex">
                <button class="button">
                  {{ item.placement }}
                </button>

                <template #portal>
                  <div>PORTAL</div>
                </template>
              </Dropdown>
            </div>
          </div>
        </div>

        <h2>Scroll vertical</h2>
        <div class="scroll-v margin-b-50">
          <div class="scroll-v__inner">
            <div v-for="(item, index) in items" :key="index">
              <h2>{{ item.title }}</h2>
              <div class="d-flex margin-b-20">
                <div v-for="(item, index) in item.items" :key="index" class="margin-h-20">
                  <Dropdown
                    :placement="item.placement"
                    mount-self
                    viewport-el=".scroll-v"
                    trigger-class="d-inline-flex"
                  >
                    <button class="button">
                      {{ item.placement }}
                    </button>

                    <template #portal>
                      <div>PORTAL</div>
                    </template>
                  </Dropdown>
                </div>
              </div>
            </div>
            <div>
              Lorem ipsum dolor sit amet consectetur, adipisicing elit.
              Saepe officiis quos vel fuga rerum perferendis similique ab molestiae?
              Dolorum odit ullam amet deserunt quos commodi? Veniam illum sapiente ipsum eum.
            </div>
          </div>
        </div>

        <div class="d-flex">
          <h2>Scroll horizontal</h2>
          <button @click="marginLeft = !marginLeft">toggle margin</button>
        </div>
        <div class="scroll-h margin-b-50">
          <div class="scroll-h__inner" :style="{ marginLeft: marginLeft ? '2000px' : '' }">
            <div v-for="(item, index) in items" :key="index">
              <h2>{{ item.title }}</h2>
              <div class="d-flex margin-b-20">
                <div v-for="(item, index) in item.items" :key="index" class="margin-h-20">
                  <Dropdown
                    :placement="item.placement"
                    mount-self
                    viewport-el=".scroll-h"
                    trigger-class="d-inline-flex"
                  >
                    <button class="button">
                      {{ item.placement }}
                    </button>

                    <template #portal>
                      <div>PORTAL</div>
                    </template>
                  </Dropdown>
                </div>
              </div>
            </div>
            <div>
              Lorem ipsum dolor sit amet consectetur, adipisicing elit.
              Saepe officiis quos vel fuga rerum perferendis similique ab molestiae?
              Dolorum odit ullam amet deserunt quos commodi? Veniam illum sapiente ipsum eum.
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Dropdown from '@/components/Dropdown.vue';

export default {
  components: {
    Dropdown,
  },
  data() {
    return {
      marginLeft: false,
      scrollPage: false,
      items: [
        {
          title: 'Start',
          items: [
            {
              placement: 'top-start|bottom-start',
            },
            {
              placement: 'right-start|left-start',
            },
            {
              placement: 'bottom-start|top-start',
            },
            {
              placement: 'left-start|right-start',
            },
          ],
        },
        {
          title: 'Center',
          items: [
            {
              placement: 'top-center|bottom-center',
            },
            {
              placement: 'right-center|left-center',
            },
            {
              placement: 'bottom-center|top-center',
            },
            {
              placement: 'left-center|right-center',
            },
          ],
        },
        {
          title: 'End',
          items: [
            {
              placement: 'top-end|bottom-end',
            },
            {
              placement: 'right-end|left-end',
            },
            {
              placement: 'bottom-end|top-end',
            },
            {
              placement: 'left-end|right-end',
            },
          ],
        },
      ],
    };
  },
};
</script>

<style lang="sass">
*
  font-family: system-ui

.page
  &--scroll
    width: 100vw
    overflow: auto

    .page__inner
      width: 3500px
      margin-left: 2500px

.wrapper
  max-width: 1100px
  margin: auto
  padding-left: 24px
  padding-right: 24px
  box-sizing: content-box

.d-inline-flex
  display: inline-flex

.d-flex
  display: flex

.margin-h-20
  margin-left: 20px
  margin-right: 20px

.margin-b-20
  margin-bottom: 20px

.margin-b-50
  margin-bottom: 50px

.button
  padding: 6px 12px
  border: 1px solid #42b883
  border-radius: 4px
  background: none
  display: inline-block
  cursor: pointer
  font-size: 16px
  line-height: 20px

.scroll-v
  height: 200px
  overflow: auto
  border: 1px solid #42b883

  &__inner
    padding: 20px
    height: 2000px

.scroll-h
  overflow: auto
  border: 1px solid #42b883

  &__inner
    padding: 20px
    width: 2000px
</style>
