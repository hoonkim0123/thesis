<template>
  <section id="s1" class="hero">
    <div class="heroImageWrap" aria-hidden="true">
      <img
        class="heroImage"
        src="/images/hero-outdoor-dining.png"
        alt=""
      >
    </div>

    <div class="heroScrim"></div>

    <div class="heroTop">
      <span class="kicker">New York Outdoor Dining</span>
      <span class="tagline">Manhattan<br>after the emergency program</span>
    </div>

    <div class="heroTitle">
      <h1 class="headline">
        <span class="line">WHY DOES</span>
        <span class="line">OUTDOOR</span>
        <span class="line">DINING</span>
        <span class="line">FEEL GONE?</span>
      </h1>
    </div>

    <div class="heroBottom">
      <p class="dek">
        Outdoor dining is still here. But it no longer appears across most everyday streets in Manhattan.
      </p>

      <p class="dek dekMuted">
        REDUCED<br>
        CONCENTRATED<br>
        CONTESTED
      </p>
    </div>
  </section>
</template>

<style scoped>
.hero {
  position: relative;
  width: 100%;
  min-height: 100svh;
  background: #101010;
  overflow: hidden;
  color: #ffffff;
}

.hero::after {
  content: "";
  position: absolute;
  left: 0;
  right: 0;
  bottom: 0;
  height: 96px;
  z-index: 5;
  pointer-events: none;
  background: linear-gradient(
    to bottom,
    rgba(0, 0, 0, 0),
    rgba(0, 0, 0, 0.68)
  );
}

.heroImageWrap {
  position: absolute;
  inset: 0;
  z-index: 0;
  overflow: hidden;
}

.heroImage {
  width: 100%;
  height: 100%;
  display: block;
  object-fit: cover;
  object-position: center center;
  transform: scale(1.06);
  filter: brightness(0.72) contrast(1.06) saturate(0.9);
  animation: imageDrift 18s ease-out forwards;
}

.heroScrim {
  position: absolute;
  inset: 0;
  z-index: 1;
  pointer-events: none;
  background:
    linear-gradient(
      to bottom,
      rgba(0, 0, 0, 0.40) 0%,
      rgba(0, 0, 0, 0.12) 34%,
      rgba(0, 0, 0, 0.34) 100%
    ),
    linear-gradient(
      to right,
      rgba(0, 0, 0, 0.64) 0%,
      rgba(0, 0, 0, 0.34) 42%,
      rgba(0, 0, 0, 0.16) 100%
    );
}

.heroTop {
  position: absolute;
  top: clamp(28px, 4vw, 58px);
  left: clamp(28px, 4vw, 72px);
  right: clamp(28px, 4vw, 72px);
  z-index: 3;
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 24px;
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: clamp(11px, 0.85vw, 14px);
  font-weight: 600;
  letter-spacing: 0.14em;
  line-height: 1.25;
  text-transform: uppercase;
}

.kicker {
  color: rgba(255, 255, 255, 0.82);
  opacity: 0;
  animation: fadeUp 0.65s ease forwards;
  animation-delay: 0.15s;
}

.tagline {
  max-width: 420px;
  text-align: right;
  color: #d85a45;
  opacity: 0;
  animation: fadeUp 0.65s ease forwards;
  animation-delay: 0.25s;
}

.heroTitle {
  position: relative;
  z-index: 2;
  min-height: 100svh;
  display: flex;
  align-items: center;
  padding: 88px clamp(28px, 4vw, 72px) 150px;
  box-sizing: border-box;
}

.headline {
  margin: 0;
  width: min(1420px, 100%);
  font-family: var(--sans, "IBM Plex Sans", sans-serif);
  font-size: clamp(58px, 9.8vw, 160px);
  font-weight: 750;
  line-height: 0.85;
  letter-spacing: -0.07em;
  text-transform: uppercase;
  color: #f7f7f4;
}

.line {
  display: block;
  opacity: 0;
  transform: translateY(30px);
  text-shadow: 0 20px 56px rgba(0, 0, 0, 0.44);
  animation: titleIn 0.8s cubic-bezier(0.2, 0.8, 0.2, 1) forwards;
}

.line:nth-child(1) {
  animation-delay: 0.35s;
}

.line:nth-child(2) {
  animation-delay: 0.48s;
}

.line:nth-child(3) {
  animation-delay: 0.61s;
}

.line:nth-child(4) {
  animation-delay: 0.74s;
}

.heroBottom {
  position: absolute;
  left: clamp(28px, 4vw, 72px);
  right: clamp(28px, 4vw, 72px);
  bottom: clamp(54px, 6vw, 88px);
  z-index: 4;
  display: grid;
  grid-template-columns: minmax(260px, 620px) minmax(220px, 420px);
  justify-content: space-between;
  align-items: end;
  gap: 40px;
}

.dek {
  margin: 0;
  font-family: var(--sans, "IBM Plex Sans", sans-serif);
  font-size: clamp(18px, 1.45vw, 25px);
  line-height: 1.32;
  font-weight: 500;
  color: rgba(255, 255, 255, 0.88);
  opacity: 0;
  animation: fadeUp 0.7s ease forwards;
  animation-delay: 1.05s;
}

.dekMuted {
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: clamp(11px, 0.9vw, 14px);
  line-height: 1.55;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  font-weight: 500;
  color: rgba(255, 255, 255, 0.72);
  text-align: right;
  animation-delay: 1.18s;
}

@keyframes titleIn {
  from {
    opacity: 0;
    transform: translateY(30px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes fadeUp {
  from {
    opacity: 0;
    transform: translateY(14px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes imageDrift {
  from {
    transform: scale(1.10);
  }

  to {
    transform: scale(1.03);
  }
}

@media (max-width: 900px) {
  .heroTop {
    align-items: flex-start;
  }

  .tagline {
    max-width: 260px;
  }

  .headline {
    font-size: clamp(54px, 14vw, 120px);
    line-height: 0.86;
    letter-spacing: -0.065em;
  }

  .heroBottom {
    grid-template-columns: 1fr;
    gap: 18px;
  }

  .dekMuted {
    text-align: left;
  }
}

@media (max-width: 620px) {
  .heroImage {
    object-position: 58% center;
  }

  .heroScrim {
    background:
      linear-gradient(
        to bottom,
        rgba(0, 0, 0, 0.68) 0%,
        rgba(0, 0, 0, 0.28) 42%,
        rgba(0, 0, 0, 0.74) 100%
      );
  }

  .heroTop {
    flex-direction: column;
    gap: 10px;
  }

  .tagline {
    text-align: left;
    max-width: 100%;
  }

  .heroTitle {
    align-items: center;
    padding-top: 118px;
    padding-bottom: 210px;
  }

  .headline {
    font-size: clamp(50px, 16vw, 88px);
  }

  .heroBottom {
    bottom: 38px;
  }

  .dek {
    font-size: 17px;
  }

  .dekMuted {
    font-size: 11px;
  }
}
</style>