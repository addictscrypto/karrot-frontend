<template>
  <div>
    <q-card class="no-shadow grey-border">
      <div
        class="edit-box"
        :class="{ changed: hasChanged }"
      >
        <form @submit.prevent="maybeSave">
          <q-field
            icon="fas fa-star"
            :label="$t('STOREEDIT.NAME')"
            :error="hasNameError"
            :error-label="nameError"
          >
            <q-input
              v-model="edit.name"
              :autofocus="true"
              autocomplete="off"
              @blur="$v.edit.name.$touch"
            />
          </q-field>
          <q-field
            icon="fas fa-handshake"
            :label="$t('STOREEDIT.STATUS')"
            :error="hasError('status')"
            :error-label="firstError('status')"
          >
            <q-select
              v-model="edit.status"
              :options="statusOptions"
            />
          </q-field>

          <q-field
            icon="fas fa-question"
            :label="$t('STOREEDIT.DESCRIPTION')"
            :error="hasError('description')"
            :error-label="firstError('description')"
          >
            <MarkdownInput :value="edit.description">
              <q-input
                v-model="edit.description"
                type="textarea"
                :min-rows="3"
                @keyup.ctrl.enter="maybeSave"
              />
            </MarkdownInput>
          </q-field>

          <q-field
            icon="fas fa-map-marker"
            :label="$t('STOREEDIT.ADDRESS')"
            :error="hasAddressError"
            :error-label="addressError"
          >
            <AddressPicker
              v-model="edit"
              :map="true"
              :color="markerColor"
              font-icon="fas fa-shopping-cart"
            />
          </q-field>

          <q-field
            icon="fas fa-calendar-alt"
            :label="$t('STOREEDIT.WEEKS_IN_ADVANCE')"
            :error="hasError('weeksInAdvance')"
            :error-label="firstError('weeksInAdvance')"
          >
            <q-slider
              v-model="edit.weeksInAdvance"
              :min="1"
              :max="10"
              label
              label-always
            />
          </q-field>

          <div
            v-if="hasNonFieldError"
            class="text-negative"
          >
            {{ firstNonFieldError }}
          </div>

          <div class="actionButtons">
            <q-btn
              type="submit"
              color="primary"
              :disable="!canSave"
              :loading="isPending"
            >
              {{ $t(isNew ? 'BUTTON.CREATE' : 'BUTTON.SAVE_CHANGES') }}
            </q-btn>
            <q-btn
              type="button"
              color="red"
              @click="archive"
              v-if="!isNew"
            >
              {{ $t('BUTTON.ARCHIVE') }}
            </q-btn>
            <q-btn
              type="button"
              @click="reset"
              v-if="!isNew"
              :disable="!hasChanged"
            >
              {{ $t('BUTTON.RESET') }}
            </q-btn>
            <q-btn
              type="button"
              @click="$emit('cancel')"
              v-if="isNew"
            >
              {{ $t('BUTTON.CANCEL') }}
            </q-btn>
          </div>
        </form>
      </div>
    </q-card>
  </div>
</template>

<script>
import { QCard, QDatetime, QField, QSlider, QOptionGroup, QInput, QBtn, QSelect, Dialog } from 'quasar'
import StandardMap from '@/components/Map/StandardMap'
import AddressPicker from '@/components/Address/AddressPicker'
import MarkdownInput from '@/components/MarkdownInput'
import { validationMixin } from 'vuelidate'
import editMixin from '@/mixins/editMixin'
import statusMixin from '@/mixins/statusMixin'
import { required, minLength, maxLength } from 'vuelidate/lib/validators'
import { statusList, optionsFor } from '@/services/storeStatus'

export default {
  name: 'StoreEdit',
  mixins: [validationMixin, editMixin, statusMixin],
  props: {
    value: {
      required: false,
      type: Object,
      default: () => ({
        name: undefined,
        description: undefined,
        weeksInAdvance: 4,
        latitude: undefined,
        longitude: undefined,
        address: undefined,
        status: 'created',
      }),
    },
    allStores: { required: true, type: Array },
  },
  components: {
    QCard, QDatetime, QField, QSlider, QOptionGroup, QInput, QBtn, QSelect, MarkdownInput, StandardMap, AddressPicker,
  },
  computed: {
    canSave () {
      if (this.$v.edit.$error) {
        return false
      }
      if (!this.isNew && !this.hasChanged) {
        return false
      }
      return true
    },
    hasNameError () {
      return !!this.nameError
    },
    nameError () {
      if (this.$v.edit.name.$error) {
        const m = this.$v.edit.name
        if (!m.required) return this.$t('VALIDATION.REQUIRED')
        if (!m.minLength) return this.$t('VALIDATION.MINLENGTH', {min: 2})
        if (!m.maxLength) return this.$t('VALIDATION.MAXLENGTH', {max: 81})
        if (!m.isUnique) return this.$t('VALIDATION.UNIQUE')
      }
      return this.firstError('name')
    },
    hasAddressError () {
      return !!this.addressError
    },
    addressError () {
      for (let field of ['address', 'latitude', 'longitude']) {
        if (this.hasError(field)) return this.firstError(field)
      }
    },
    statusOptions () {
      return statusList
        .filter(s => s.selectable)
        .map(s => ({
          value: s.key,
          label: this.$t(s.label),
          leftColor: s.color,
          icon: s.icon,
        }))
    },
    markerColor () {
      if (this.edit) return optionsFor(this.edit).color
    },
  },
  methods: {
    maybeSave () {
      this.$v.edit.$touch()
      if (!this.canSave) return
      this.save()
    },
    archive (event) {
      Dialog.create({
        title: this.$t('STOREEDIT.DIALOGS.ARCHIVE.TITLE'),
        message: this.$t('STOREEDIT.DIALOGS.ARCHIVE.MESSAGE'),
        cancel: this.$t('BUTTON.CANCEL'),
        ok: this.$t('STOREEDIT.DIALOGS.ARCHIVE.CONFIRM'),
      })
        .then(() => this.$emit('save', { id: this.value.id, status: 'archived' }, event))
        .catch(() => {})
    },
  },
  validations: {
    edit: {
      name: {
        required,
        minLength: minLength(3),
        maxLength: maxLength(80),
        isUnique (value) {
          if (value === '') return true
          return this.allStores
            .filter(e => e.id !== this.edit.id)
            .findIndex(e => e.name === value) < 0
        },
      },
    },
  },
}
</script>

<style scoped lang="stylus">
@import '~editbox'
</style>
