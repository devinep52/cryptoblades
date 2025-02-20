<template>
  <div>
    <h2 class="pt-3">{{ promoQuestTemplates ? $t('quests.promoQuestTemplates') : $t('quests.questTemplates') }}</h2>
    <b-form-group class="m-3">
      <b-form-radio v-model="promoQuestTemplates" @change="refreshQuestTemplates" :value="false">
        {{ $t('quests.questsTitle') }}
      </b-form-radio>
      <b-form-radio v-model="promoQuestTemplates" @change="refreshQuestTemplates" :value="true">
        {{ $t('quests.promoQuests') }}
      </b-form-radio>
    </b-form-group>
    <b-form inline>
      <label class="mr-sm-2">{{ $t('quests.templatesTier') }}</label>
      <b-form-select class="mt-2 mb-2" v-model="templatesTier" @change="refreshQuestTemplates">
        <b-form-select-option :value="undefined" disabled>
          {{ $t('quests.pleaseSelectQuestTier') }}
        </b-form-select-option>
        <b-form-select-option v-for="tier in tiers" :key="tier" :value="tier">
          {{ $t(`quests.rarityType.${Rarity[tier]}`) }}
        </b-form-select-option>
      </b-form-select>
    </b-form>
    <QuestsList v-if="templatesTier !== undefined" :tier="promoQuestTemplates ? templatesTier + 10 : templatesTier"
                deletable/>
  </div>
</template>

<script lang="ts">
import Vue from 'vue';
import {mapActions} from 'vuex';
import {Quest, Rarity} from '@/views/Quests.vue';
import QuestsList from './QuestsList.vue';

interface StoreMappedActions {
  getQuestTemplates(payload: { tier: number }): Promise<Quest[]>;

  getQuestDeadline(payload: { questID: number }): Promise<number>;

  getQuestSupply(payload: { questID: number }): Promise<number>;
}

interface Data {
  tiers: Rarity[];
  questTemplates: Quest[];
  templatesTier?: Rarity;
  promoQuestTemplates: boolean;
  isLoading: boolean;
}

export default Vue.extend({

  components: {QuestsList},

  data() {
    return {
      tiers: [Rarity.COMMON, Rarity.UNCOMMON, Rarity.RARE, Rarity.EPIC, Rarity.LEGENDARY],
      questTemplates: [],
      templatesTier: undefined,
      promoQuestTemplates: false,
      isLoading: false,
      Rarity,
    } as Data;
  },

  methods: {
    ...mapActions([
      'getQuestTemplates',
      'getQuestDeadline',
      'getQuestSupply',
    ]) as StoreMappedActions,

    async refreshQuestTemplates() {
      if (this.templatesTier === undefined) return;
      try {
        this.isLoading = true;
        if (this.promoQuestTemplates) {
          const promoTier = this.templatesTier + 10;
          this.questTemplates = await this.getQuestTemplates({tier: promoTier});
        } else {
          this.questTemplates = await this.getQuestTemplates({tier: this.templatesTier});
        }
        for (const quest of this.questTemplates) {
          quest.deadline = +await this.getQuestDeadline({questID: quest.id});
          quest.supply = +await this.getQuestSupply({questID: quest.id});
        }
      } finally {
        this.isLoading = false;
      }
    },
  },

  async mounted() {
    await this.refreshQuestTemplates();
    this.$root.$on('refresh-quest-templates', async () => {
      await this.refreshQuestTemplates();
    });
  }

});
</script>

<style scoped>
</style>
