<template>
  <div class="container">
    <div v-if="imgCounter === null" class="initial-state">
      <h1>HOT SAMPLES in your area!</h1>
      <img class="default-img" :src="hottieImg" alt="Placeholder Image" /><br>
      <button @click="setImgCounter" class="btn">See the hotties</button>
    </div>

    <div v-else-if="imgCounter <= 11" class="image-picker">
      <h2>Which hottie do you prefer?</h2>
      <div class="image-options">
        <img
          v-for="(img, index) in images"
          :key="index"
          :src="img"
          @click="selectImage(index)"
          class="selectable-img"
          :alt="'Image ' + (index + 1)"
        />
      </div>
    </div>
    <div v-else>
      <h1>Wow... So that's your type?!</h1>
      <h2>Based on our advanced AI algorithm, you'll enjoy having these samples at home:</h2>
      <div v-for="(colors, groupType) in suggestions" :key="groupType">
        <h3>{{ groupType }}</h3>
      <img
        v-for="(color, index) in colors"
        :key="index"
        :src="`https://res.cloudinary.com/cstm/image/upload/v1/common/swatch/${color}/small/A.svg`"
        class="swatch-with-border"
        :alt="color"/>
      </div>
    </div>
  </div>
</template>

<script>
import colorMap from '../helpers/colorMap';
import availableColors from '../helpers/availableColors';

export default {
  data() {
    const allImages = Array.from({ length: 34 }, (_, index) => index);
    const shuffledImages = [...allImages].sort(() => Math.random() - 0.5);
    return {
      imgCounter: null, 
      favTypes: {},
      favColors: {},
      allImages: shuffledImages,
    };
  },
  methods: {
    setImgCounter() {
      this.imgCounter = 0;
    },
    selectImage(index) {
      const selected = this.allImages[2 * this.imgCounter + index];
      const item = colorMap[selected];
      const [shelfType, shelfColor] = item.split('_');  // Split the string into two parts

      // Count occurrences for the part before _
      if (this.favTypes[shelfType]) {
        this.favTypes[shelfType]++;
      } else {
        this.favTypes[shelfType] = 1;
      }

      // Count occurrences for the part after _
      if (this.favColors[shelfColor]) {
        this.favColors[shelfColor]++;
      } else {
        this.favColors[shelfColor] = 1;
      }
      this.imgCounter += 1;
    },
  },
  computed: {
    hottieImg() {
      const hottieId = Math.random() < 0.5 ? 1 : 2;
      return `/hottie${hottieId}.png`;
    },
    images() {
      const img1 = this.allImages[this.imgCounter * 2];
      const img2 = this.allImages[this.imgCounter * 2 + 1];
      return [`/photos/img_${img1}.jpg`, `/photos/img_${img2}.jpg`];
    },
    suggestions() {
      const group = {
        'original': [],
        'tone': [],
        'edge': [],
      }
      const typeToGroup = {
        'T01': 'original',
        'T02': 'original',
        'T01v': 'original',
        'T03': 'tone',
        'T13': 'edge',
      }
      const bestColor = Object.entries(this.favColors).sort((a, b) => b[1] - a[1])[0][0];
      const bestType = Object.entries(this.favTypes).sort((a, b) => b[1] - a[1])[0][0];
      const tops = availableColors[bestType][bestColor];
      group[typeToGroup[bestType]] = tops;

      const secondBestType = Object.entries(this.favTypes).sort((a, b) => b[1] - a[1])[1][0];
      const secondTops = availableColors[secondBestType][bestColor];
      group[typeToGroup[secondBestType]] = [...group[typeToGroup[secondBestType]], ...secondTops];
      
      const otherImportantType = {
        'T01': 'T02',
        'T02': 'T01',
        'T03': 'T13',
        'T13': 'T03',
        'T01v': 'T01',
      }[bestType];
      const otherTops = availableColors[otherImportantType][bestColor];
      group[typeToGroup[otherImportantType]] = [...group[typeToGroup[otherImportantType]], ...otherTops];
      
      const returnable = {};
      const bestGroup = typeToGroup[bestType];
      returnable[bestGroup] = group[bestGroup].slice(0, 5);
      if (group[bestGroup].length < 5) {
        const remaining = 5 - group[bestGroup].length;
        const secondGroup = Object.entries(typeToGroup).find(([type, group]) => type !== bestType && group !== bestGroup)[0];
        returnable[secondGroup] = group[secondGroup].slice(0, remaining);
      }

      return returnable;
    }
  }
};
</script>

<style scoped>
/* Container styles */
.container {
  text-align: center;
  margin: 50px auto;
  padding: 20px;
  max-width: 1400px;
  background-color: #f7f7f7;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

/* Initial state styles */
.initial-state {
  padding: 20px;
}

.default-img {
  margin: 20px auto;
  max-width: 640px;
  height: auto;
}

.btn {
  padding: 10px 20px;
  background-color: #3498db;
  color: white;
  border: none;
  border-radius: 5px;
  font-size: 16px;
  cursor: pointer;
}

.btn:hover {
  background-color: #2980b9;
}

/* Image picker styles */
.image-picker h2 {
  margin-bottom: 20px;
}

.image-options {
  display: flex;
  justify-content: center;
  gap: 20px;
}

.selectable-img {
  width: 640px;
  height: 640px;
  border: 2px solid transparent;
  cursor: pointer;
  transition: border-color 0.3s;
}

.selectable-img:hover {
  border-color: #2ecc71;
}

.swatch-with-border {
  width: 50px;
  height: 50px;
  border: 2px solid #000000;
  border-radius: 50%;
  margin: 10px;
}
</style>
