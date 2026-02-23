---
layout: ../../layouts/RawLayout.astro
title: "Alkaline Mixtape Vol. 1"
date: "2025-11-17"
---

# Alkaline Mixtape Vol. 1

> Date: Nov 2025
>
> Assembled from collective recordings: personal sounds, field fragments, live documents, and machine noise. The materials were processed and re-arranged into a continuous listen: a small archive of [Southern Alkaline](https://www.instagram.com/southern.alkaline/) as a living system.

<iframe width="100%" height="166" scrolling="no" frameborder="no" allow="autoplay" src="https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/soundcloud%253Atracks%253A2247073928&color=%23ff5500&auto_play=false&hide_related=false&show_comments=true&show_user=true&show_reposts=false&show_teaser=true&visual=false"></iframe><div style="font-size: 10px; color: #cccccc;line-break: anywhere;word-break: normal;overflow: hidden;white-space: nowrap;text-overflow: ellipsis; font-family: Interstate,Lucida Grande,Lucida Sans Unicode,Lucida Sans,Garuda,Verdana,Tahoma,sans-serif;font-weight: 100;"><a href="https://soundcloud.com/alkaline-598972056" title="Southern Alkaline" target="_blank" style="color: #cccccc; text-decoration: none;">Southern Alkaline</a> · <a href="https://soundcloud.com/alkaline-598972056/alkaline-mixtape-vol1" title="Alkaline Mixtape Vol.1" target="_blank" style="color: #cccccc; text-decoration: none;">Alkaline Mixtape Vol.1</a></div>

<br>
This mixtape began as a simple call for sounds: 1–3 recordings per person, no rules, only a willingness to send something from life, even if it’s “just noise.” The responses formed a messy collective dataset: intimate everyday recordings, performance documents, rough sketches, device hums, rain, gestures, glitches...

I processed the files and composed the mixtape, using [**Fluid Corpus Manipulation**](https://www.flucoma.org/) for feature-based slicing and selection (MFCC + spectral descriptors), plus probabilistic thinning (“dice”) to keep the dataset breathable. The final piece moves through a set of loosely-defined chapters: each file first treated as a single point (macro layout), then re-entered as grained slices for collage (micro texture).

Composition here is a negotiation with an over-abundant corpus. I used **SuperCollider + FluCoMa** to slice audio by onset metrics, extract MFCC and spectral-shape statistics, and filter materials by loudness and probability. Instead of seeking an ideal representative sample, I leaned into thinning, loss, and partiality, reducing tens of thousands of slices into a few thousand points that can actually breathe from massive noise cloud.

The structure is hierarchical (作为诅咒的必要解码方式): at the top level, each file becomes a point in a small map to decide when it enters (chapter layout); at the second level, selected files are opened again as a dense 2D field for collage. What you hear is a temporary decision: a “rest sign” drawn on something that could continue to branch forever.


## Sounds contributed by
*[ajiao](https://www.instagram.com/ajiaoajiaoajiaoajiao/), huazi, [keyu](https://www.instagram.com/keyu_u_u/), [Liu Hanwen](https://www.instagram.com/liuhanwen9918/), [Peng](https://www.instagram.com/suanmuzhipeng/), [Li Xiangwei](https://www.instagram.com/lixiangwei_/), [ResBina](https://www.instagram.com/res.binaa/), [stevelongoriaa](https://www.instagram.com/stevelongoriaa/), [xuanni](https://www.instagram.com/xuanni_space/), [xiuyuan](https://www.instagram.com/yuanxiushen/), [yutong](https://www.instagram.com/gaystaatspo/), [yz](https://www.instagram.com/aiisisssiaaais/)*  

<details>
  <summary><strong>source list</strong></summary>

  <!-- Option A: 人名 + 一句素材描述（更好看） -->
  - ajiao — 风力发电旋转 小中甸服务区加水站, 牧民房屋顶早晨融水, 1021羊跳海(39-56);
  - huazi — 雨偷(触发电瓶车报警器), 雨打在伞上, Overtrip3 Live Recording;
  - keyu - Yang playing violin[2, 13, 16, 52, 62];
  - Liu Hanwen - 电扇[0a4e8576e4f7d...28a7a15874_raw, 98dd6ebe26ade...02c62ee021_raw];
  - Li Xiangwei - 20230618 沙丘演出;
  - Peng - 8.10 轻松听, 弹, 独白, 鞋底掉了;
  - ResBina - 14-Audio 000...85510;
  - stevelongoriaa - darkmass, maggotcrucifix;
  - xuanni - Wales Coast to Coast;
  - xiuyuan - 20250910, 12, SC_251105_030732;
  - yutong — druming glicth, 声像故障;
  - yz - 新录音[52, 50, 46, 71];
</details>
<br>
<br>

## UMAP
<div class="gallery-container">
  <figure class="figure-card">
    <img src="/images/alkaline-mixtape-2dmap.png" alt="A black-and-white 2D scatter plot of thousands of points forming dense clusters and voids, representing audio slices projected by timbral similarity." class="lightbox-trigger" />
    <figcaption>
      <strong>FluCoMa 2D Map | Corpus projection</strong><br/>
      Each dot represents a short audio slice extracted from the submitted recordings. For every slice, timbral descriptors (MFCC and spectral-shape statistics)  were computed and the high-dimensional feature vectors were projected into 2-dimension using UMAP. Nearby points indicate similar timbral profiles; dense regions suggest recurrent textures across the corpus. This map was used as a navigation surface to select and re-arrange slices into the final collage.
    </figcaption>
  </figure>

  <figure class="figure-card">
    <img src="/images/alkaline-mixtape-2dmap2.png" alt="A second black-and-white 2D corpus map with a different distribution of clusters, showing another projection or parameter setting." class="lightbox-trigger" />
    <figcaption>
      <strong>Different density and filtering | parameter setting</strong><br/>
      A black-and-white 2D plot of ten thousands of points forming dense clusters and voids, representing audio slices projected by timbral similarity.
    </figcaption>
  </figure>
</div>

<br>
<br


  ### Acoustic-Topography | Islands & Homogeneous Cloud Masses

  Within the chain of slicing → descriptors → embedding → projection, traces are produced. Dots and trajectories appear on the screen. I explore this field and look for the signs of signal.

  The map offers a spatialized mode of listening and navigation. It places field recordings, live fragments, everyday noise, and synthetic material into the same metric system, letting aggregation and spillover surface as topographic phenomena: islands, ridge lines, fault lines, cloud masses. Dragging the mouse through it feels like walking inside a computed sound field; changes in parameters and scale shift the resolution of “similarity,” reshaping which sounds become perceptually adjacent.

  As the material becomes noisier, more random, and harder to name with stable differences, the point cloud does not behave like the taxonomy-like automatic machine I initially expected. It renders sonic difference and uncertainty in spatial form, functioning like an instrument that records noise, memory, and semantic shards at once.

  Slices that carry a clearer contour of sonic events tend to form islands and boundaries, with legible textures preserved inside those borders. When multiple noise sites are sliced into countless fragments of only a few seconds each, the material appears more uniform in texture, with fewer nameable forms, condensing into dense sandbox-like cloud masses. The system reads descriptor vectors and their distance relations; this diverges from embodied experience on site. A body arrives with memory, sensation, context, expectation, and fatigue; algorithmic slicing arrives with spectrum, energy, loudness, and dynamic contours. Two differently encoded layers of experience / information / data run in parallel on the map: one organized by narrative, one by statistics. They illuminate each other, and they also fall out of alignment.

  In this mixtape, due to the toolchain’s machine listening / machine learning character, the compositional aspect emerges as a shift in organizational principles: from the ear’s intuitive sorting toward the measurement and visualization of feature space. Noise participates in differentiation while continuously producing deviation. A vast interzone persists between noise, meaning, and remainder; it remains shareable, waiting to be revisited, to sediment, and to flow back.

<br>
<br>

## Cover

For the cover image, [**ajiao**](https://www.instagram.com/ajiaoajiaoajiaoajiao/) approached it as an experiment rather than a task: trying methods without a fixed goal, then letting accidents suggest the next decision. He described the hardest part as judging an abstract object without a clear standard: deciding when to stop, or choosing one outcome among many. The final cover is a typographic extraction from the project’s file names, iterated through multiple versions and adjusted through back-and-forth.
<br>

![Alkaline Mixtape Cover](/images/alkaline-mixtape-cover.png)
<br>

## Extended Mix
Southern Alkaline member [**Yutong**](https://www.instagram.com/gaystaatspo/) later made an extended mix ([freebasing coded 1](https://drive.google.com/file/d/1NKdPvBWkvpB__xQxLebHTDB7B0vT33Cr/view?usp=drivesdk), 1h06m), which folds this mixtape into its second half and was described as "pure madness".
<br>
<br>

<!-- Lightbox Structure & Styles -->
<div id="lightbox" class="lightbox">
  <span class="close" id="lightbox-close">&times;</span>
  <img class="lightbox-content" id="lightbox-img">
  <div id="caption"></div>
</div>

<style>
  .gallery-container {
    display: flex;
    gap: 20px;
    flex-wrap: wrap;
    align-items: flex-start;
  }
  .figure-card {
    flex: 1;
    min-width: 300px;
    margin: 0;
    border: 1px solid #333;
    padding: 15px;
    background: #0a0a0a;
  }
  .figure-card img {
    width: 100%;
    height: auto;
    cursor: zoom-in;
    border: 1px solid #222;
    transition: border-color 0.3s;
  }
  .figure-card img:hover {
    border-color: #0f0;
  }
  .figure-card figcaption {
    margin-top: 15px;
    padding-top: 10px;
    border-top: 1px dotted #333;
    font-size: 0.85rem;
    color: #888;
    font-family: monospace;
    line-height: 1.4;
  }
  /* Lightbox */
  .lightbox { display: none; position: fixed; z-index: 9999; left: 0; top: 0; width: 100%; height: 100%; overflow: auto; background-color: rgba(0,0,0,0.9); backdrop-filter: blur(5px); }
  .lightbox-content { margin: auto; display: block; width: 80%; max-width: 1200px; max-height: 90vh; object-fit: contain; border: 1px solid #0f0; margin-top: 5vh; }
  #caption { margin: auto; display: block; width: 80%; max-width: 700px; text-align: center; color: #ccc; padding: 10px 0; height: 150px; font-family: monospace; margin-top: 10px; }
  .close { position: absolute; top: 20px; right: 35px; color: #f1f1f1; font-size: 40px; font-weight: bold; transition: 0.3s; cursor: pointer; }
  .close:hover, .close:focus { color: #0f0; text-decoration: none; cursor: pointer; }
</style>

<script>
  const lightbox = document.getElementById('lightbox');
  const lightboxImg = document.getElementById('lightbox-img');
  const captionText = document.getElementById('caption');
  const closeBtn = document.getElementById("lightbox-close");

  document.querySelectorAll('.lightbox-trigger').forEach(img => {
    img.onclick = function(){
      lightbox.style.display = "block";
      lightboxImg.src = this.src;
      captionText.innerHTML = this.alt;
    }
  });

  closeBtn.onclick = function() { lightbox.style.display = "none"; }
  lightbox.onclick = function(e) { if(e.target !== lightboxImg) lightbox.style.display = "none"; }
  document.addEventListener('keydown', function(event) { if (event.key === "Escape") lightbox.style.display = "none"; });
</script>
<br>
<br>


<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>