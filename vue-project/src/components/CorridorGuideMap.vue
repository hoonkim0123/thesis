<script setup>
import { computed, ref } from 'vue'

const BASE_URL = import.meta.env.BASE_URL

const streets = [
  {
    name: 'Amsterdam Avenue',
    shortName: 'Amsterdam',
    area: 'Upper West Side',
    line: 'Repeated setups remain close enough to feel visible.',
    images: [
      {
        id: 'amsterdam-corridor',
        type: 'Street view',
        restaurant: 'The Wolfe · Momoya',
        caption: 'Repeated setups remain visible along the avenue.',
        base: '/images/s8/amsterdam/amsterdam_corridor_the-wolfe-momoya_angle_visible_001.jpg',
        mask: '',
      },
      {
        id: 'amsterdam-salumeria',
        type: 'Specific setup',
        restaurant: 'Salumeria Rosi',
        caption: 'A single setup becomes part of the street edge.',
        base: '/images/s8/amsterdam/amsterdam_283_salumeria-rosi_detail_visible_001.jpg',
        mask: '',
      },
      {
        id: 'amsterdam-celeste',
        type: 'Detail',
        restaurant: 'Celeste',
        caption: 'The structure is visible as its own object in the roadway.',
        base: '/images/s8/amsterdam/amsterdam_502_celeste_detail_visible_001.jpg',
        mask: '',
      },
    ],
  },
  {
    name: 'Columbus Avenue',
    shortName: 'Columbus',
    area: 'Upper West Side',
    line: 'Outdoor dining appears in separated pockets.',
    images: [
      {
        id: 'columbus-felice',
        type: 'Street view',
        restaurant: 'Felice',
        caption: 'A visible setup remains at the corner.',
        base: '/images/s8/columbus/columbus_240_felice_wide_visible_001.jpg',
        mask: '',
      },
      {
        id: 'columbus-la-pecora',
        type: 'Specific setup',
        restaurant: 'La Pecora Bianca',
        caption: 'The remaining structure is large, but the corridor still feels spaced out.',
        base: '/images/s8/columbus/columbus_359_la-pecora-bianca_detail_visible_001.jpg',
        mask: '',
      },
      {
        id: 'columbus-pocket',
        type: 'Pocket',
        restaurant: 'Columbus Avenue',
        caption: 'Outdoor dining appears as pockets rather than a continuous line.',
        base: '/images/s8/columbus/columbus_unknown_turquoise-shed_angle_visible_001.jpg',
        mask: '',
      },
    ],
  },
  {
    name: '2nd Avenue',
    shortName: '2nd Ave',
    area: 'Upper East Side',
    line: 'Outdoor dining appears block by block.',
    images: [
      {
        id: 'second-la-pecora',
        type: 'Street view',
        restaurant: 'La Pecora Bianca',
        caption: 'Visible, but not continuous.',
        base: '/images/s8/second-ave/2nd-ave_1562_la-pecora-bianca_wide_visible_001.jpg',
        mask: '',
      },
      {
        id: 'second-cafe-maud',
        type: 'Corner',
        restaurant: 'Cafe Maud',
        caption: 'A corner setup marks one block, then the pattern breaks.',
        base: '/images/s8/second-ave/2nd-ave_132_cafe-maud_wide_visible_001.jpg',
        mask: '',
      },
      {
        id: 'second-boqueria',
        type: 'Detail',
        restaurant: 'Boqueria',
        caption: 'Smaller sidewalk setups make the pattern feel more scattered.',
        base: '/images/s8/second-ave/2nd-ave_boqueria_detail_visible_001.jpg',
        mask: '',
      },
    ],
  },
  {
    name: 'Mulberry Street',
    shortName: 'Mulberry',
    area: 'Little Italy',
    line: 'Outdoor dining still reads as part of the street.',
    images: [
      {
        id: 'mulberry-corridor',
        type: 'Street view',
        restaurant: 'Da Gennaro · La Mela',
        caption: 'Here, outdoor dining still feels like part of the street itself.',
        base: '/images/s8/mulberry/mulberry_corridor_da-gennaro-la-mela_wide_visible_001.jpg',
        mask: '',
      },
      {
        id: 'mulberry-repeat',
        type: 'Repeated presence',
        restaurant: 'La Mela · Amici',
        caption: 'The setups repeat closely enough to become a street identity.',
        base: '/images/s8/mulberry/mulberry_167_la-mela_detail_visible_001.jpg',
        mask: '',
      },
      {
        id: 'mulberry-grotta',
        type: 'Specific setup',
        restaurant: 'Grotta Azzurra',
        caption: 'A named restaurant makes the remaining pattern specific.',
        base: '/images/s8/mulberry/mulberry_177_grotta-azzurra_detail_visible_001.jpg',
        mask: '',
      },
    ],
  },
]

const activeStreetIndex = ref(0)
const activeImageIndex = ref(0)

const activeStreet = computed(() => streets[activeStreetIndex.value])
const activeImage = computed(() => activeStreet.value.images[activeImageIndex.value])

function resolveAsset(path) {
  if (!path) return ''
  return `${BASE_URL}${String(path).replace(/^\//, '')}`
}

function selectStreet(index) {
  activeStreetIndex.value = index
  activeImageIndex.value = 0
}

function selectImage(index) {
  activeImageIndex.value = index
}
</script>

<template>
  <div class="s8-photo-essay">
    <div class="s8-picker" aria-label="Street selector">
      <button
        v-for="(street, index) in streets"
        :key="street.name"
        class="s8-street"
        :class="{ 'is-active': activeStreetIndex === index }"
        type="button"
        @click="selectStreet(index)"
      >
        <span>{{ street.shortName }}</span>
        <small>{{ street.area }}</small>
      </button>
    </div>

    <div class="s8-stage">
      <div class="s8-photo-wrap">
        <img
          class="s8-photo"
          :src="resolveAsset(activeImage.base)"
          :alt="`${activeStreet.name}, ${activeImage.restaurant}`"
        />

        <img
          v-if="activeImage.mask"
          class="s8-mask"
          :src="resolveAsset(activeImage.mask)"
          alt=""
          aria-hidden="true"
        />

        <div class="s8-photo-text">
          <div class="s8-meta">
            <span>{{ activeImage.type }}</span>
            <span>{{ activeImage.restaurant }}</span>
          </div>

          <p>{{ activeImage.caption }}</p>
        </div>
      </div>

      <div class="s8-lower">
        <div class="s8-caption">
          <div class="s8-caption-kicker">Street level</div>
          <h2>{{ activeStreet.name }}</h2>
          <p>{{ activeStreet.line }}</p>
        </div>

        <div class="s8-thumbs" aria-label="Photo selector">
          <button
            v-for="(image, index) in activeStreet.images"
            :key="image.id"
            class="s8-thumb"
            :class="{ 'is-active': activeImageIndex === index }"
            type="button"
            @click="selectImage(index)"
          >
            <span class="s8-thumb-img">
              <img
                :src="resolveAsset(image.base)"
                :alt="`${image.type}, ${image.restaurant}`"
              />
            </span>

            <span class="s8-thumb-copy">
              <span>{{ image.type }}</span>
              <small>{{ image.restaurant }}</small>
            </span>
          </button>
        </div>
      </div>

      <p class="s8-note">
        Photos were taken after the March 2026 dataset snapshot and are used as field observations.
      </p>
    </div>
  </div>
</template>

<style scoped>
.s8-photo-essay {
  width: 100%;
  display: grid;
  grid-template-columns: 190px minmax(0, 1fr);
  gap: 42px;
  align-items: start;
}

.s8-picker {
  position: sticky;
  top: 96px;
  display: grid;
  gap: 2px;
  padding-top: 6px;
}

.s8-street {
  width: 100%;
  border: 0;
  border-left: 3px solid transparent;
  background: transparent;
  padding: 14px 0 14px 16px;
  text-align: left;
  cursor: pointer;
  color: var(--muted, #555);
  transition:
    border-color 0.15s ease,
    color 0.15s ease,
    background 0.15s ease;
}

.s8-street:hover {
  background: rgba(0, 0, 0, 0.025);
  color: var(--ink, #111);
}

.s8-street.is-active {
  border-left-color: var(--accent, #b44a3c);
  color: var(--ink, #111);
  background: rgba(180, 74, 60, 0.055);
}

.s8-street span {
  display: block;
  font-family: var(--sans, "IBM Plex Sans", sans-serif);
  font-size: 17px;
  line-height: 1.15;
  font-weight: 700;
  letter-spacing: -0.02em;
}

.s8-street small {
  display: block;
  margin-top: 5px;
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 10px;
  line-height: 1.3;
  color: var(--ghost, #999);
}

.s8-stage {
  min-width: 0;
}

.s8-photo-wrap {
  position: relative;
  width: 100%;
  aspect-ratio: 16 / 9;
  overflow: hidden;
  background: #e9e6df;
}

.s8-photo {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  display: block;
  object-fit: cover;
  object-position: center;
}

.s8-mask {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  display: block;
  object-fit: cover;
  object-position: center;
  pointer-events: none;
}

.s8-photo-text {
  position: absolute;
  left: 0;
  right: 0;
  bottom: 0;
  padding: 88px 32px 28px;
  background: linear-gradient(
    to top,
    rgba(17, 17, 17, 0.72),
    rgba(17, 17, 17, 0.26),
    rgba(17, 17, 17, 0)
  );
  color: #fff;
}

.s8-meta {
  display: flex;
  justify-content: space-between;
  gap: 24px;
  margin-bottom: 12px;
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 11px;
  line-height: 1.35;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: rgba(255, 255, 255, 0.72);
}

.s8-photo-text p {
  max-width: 720px;
  margin: 0;
  font-family: var(--sans, "IBM Plex Sans", sans-serif);
  font-size: clamp(24px, 2.4vw, 38px);
  line-height: 1.08;
  letter-spacing: -0.045em;
  font-weight: 700;
  color: rgba(255, 255, 255, 0.96);
}

.s8-lower {
  display: grid;
  grid-template-columns: 360px minmax(0, 1fr);
  gap: 42px;
  align-items: start;
  margin-top: 24px;
}

.s8-caption-kicker {
  margin-bottom: 10px;
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 10px;
  line-height: 1.3;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  color: var(--ghost, #999);
}

.s8-caption h2 {
  margin: 0 0 10px;
  font-family: var(--sans, "IBM Plex Sans", sans-serif);
  font-size: clamp(28px, 3vw, 48px);
  line-height: 0.98;
  letter-spacing: -0.06em;
  font-weight: 750;
  color: var(--ink, #111);
}

.s8-caption p {
  max-width: 330px;
  margin: 0;
  font-family: var(--sans, "IBM Plex Sans", sans-serif);
  font-size: 16px;
  line-height: 1.45;
  color: var(--muted, #555);
}

.s8-thumbs {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 12px;
}

.s8-thumb {
  display: grid;
  gap: 10px;
  border: 0;
  background: transparent;
  padding: 0;
  text-align: left;
  cursor: pointer;
  opacity: 0.58;
  transition:
    opacity 0.15s ease,
    transform 0.15s ease;
}

.s8-thumb:hover {
  opacity: 0.9;
}

.s8-thumb.is-active {
  opacity: 1;
}

.s8-thumb-img {
  display: block;
  width: 100%;
  aspect-ratio: 4 / 3;
  overflow: hidden;
  background: #e9e6df;
}

.s8-thumb-img img {
  width: 100%;
  height: 100%;
  display: block;
  object-fit: cover;
}

.s8-thumb.is-active .s8-thumb-img {
  outline: 3px solid var(--accent, #b44a3c);
  outline-offset: 0;
}

.s8-thumb-copy span {
  display: block;
  font-family: var(--sans, "IBM Plex Sans", sans-serif);
  font-size: 14px;
  line-height: 1.2;
  font-weight: 700;
  color: var(--ink, #111);
}

.s8-thumb-copy small {
  display: block;
  margin-top: 3px;
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 10px;
  line-height: 1.3;
  color: var(--ghost, #999);
}

.s8-note {
  margin: 26px 0 0;
  padding-top: 16px;
  border-top: 1px solid var(--rule, #e6e6e6);
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 10px;
  line-height: 1.5;
  color: var(--ghost, #999);
}

@media (max-width: 1000px) {
  .s8-photo-essay {
    grid-template-columns: 1fr;
    gap: 22px;
  }

  .s8-picker {
    position: static;
    display: flex;
    overflow-x: auto;
    gap: 8px;
    padding: 0 0 4px;
  }

  .s8-street {
    min-width: 160px;
    border-left: 0;
    border-bottom: 3px solid transparent;
    padding: 12px 12px 14px;
  }

  .s8-street.is-active {
    border-bottom-color: var(--accent, #b44a3c);
    border-left-color: transparent;
  }

  .s8-lower {
    grid-template-columns: 1fr;
  }

  .s8-caption p {
    max-width: 560px;
  }
}

@media (max-width: 680px) {
  .s8-photo-wrap {
    aspect-ratio: 4 / 5;
  }

  .s8-photo-text {
    padding: 82px 20px 22px;
  }

  .s8-meta {
    flex-direction: column;
    gap: 4px;
  }

  .s8-thumbs {
    grid-template-columns: 1fr;
  }

  .s8-thumb {
    grid-template-columns: 96px 1fr;
    align-items: center;
  }

  .s8-thumb-img {
    aspect-ratio: 4 / 3;
  }
}
</style>