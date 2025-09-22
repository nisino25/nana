<template>
  <div 
    class="gameCard" 
    :id="'card-'+card?.id" 
    :class="{ flipped: card.flipped}"  
    @click="$emit('flip', card.id)"
  >

    <div class="card-inner">
        <div class="card-front">
          <span>{{ card?.number }}</span>
        </div>
        <div class="card-back"></div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    card: {
      type: Object,
      required: true
    }
  },
  emits: ['flip']
};
</script>
