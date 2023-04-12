<template>
  <h1 class="title">MMFIN</h1>
  <section class="container">
    <div class="myDiv" ref="canvasContainer">
      <canvas ref="canvas"></canvas>
    </div>
    <div class="palette">
      <div class="item" @click="getClipboard(backgroundColor)" v-for="(backgroundColor, index) in myBackground" :key="index" :style="{backgroundColor, color: myText[index]}">
        {{ backgroundColor }}
      </div>
    </div>
  </section>

</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
//캔버스 컨테이너 변수(div)
const canvasContainer = ref<HTMLDivElement | null>(null);
//캔버스 요소 변수
const canvas = ref<HTMLCanvasElement | null>(null);
//아이템 배경색 변수 
const myBackground = ref<string[]>([]);
//아이템 글자색 변수
const myText = ref<string[]>([]);
//이미지
const img = new Image();

const getClipboard = (color: string) => {
  navigator.clipboard.writeText(color)
}

onMounted(() => {
  if (canvas.value && canvasContainer.value) {
    const ctx: any = canvas.value.getContext('2d', {willReadFrequently: true});

    const resizeCanvas = () => {
      if(canvas.value){
        canvas.value.width = window.innerWidth - 200;
        canvas.value.height = window.innerHeight;
        drawImage(ctx, canvas.value, img);
      }
    };

    // 초기 사이즈 설정
    resizeCanvas();

    // 화면 크기가 변경될 때마다 캔버스 크기 다시 계산
    window.addEventListener('resize', resizeCanvas);

    canvas.value.addEventListener('click', (event :any) => {
      if(img.src){
        const x = event.offsetX;
        const y = event.offsetY;
        //캔버스 이미지 가져오기
        const pixel = ctx.getImageData(x, y, 1, 1).data;
        

        /*
        밝기(brightness)를 계산하는 코드
        각 색상 채널(R, G, B)은 0부터 255 사이의 값을 가질 수 있음
        여기서 299, 587, 114는 각 R,G,B 채널 밝기에 미치는 가중치임
        이 값은 인간의 시각 특성을 고려하여 결정된 값으로, 반영된 숫자임

        각 색상 채널 밝기 값을 구하려면, 각 색상 채널 값을 가중치와 함께 곱한 후, 총 합을 구하면 됨
        이렇게 구한 총합을 1000으로 나누면 0~255사이의 값을 가지는 밝기 값(brightness)이 계산됨
        */
        const brightness: number = Math.round((
          pixel[0] * 299 +
          pixel[1] * 587 +
          pixel[2] * 114
        ) / 1000);
        //배경 색 지정
        myBackground.value.push(`rgba(${pixel[0]}, ${pixel[1]}, ${pixel[2]}, ${pixel[3]})`)
        //글자 색 지정
        myText.value.push((brightness > 125) ?  '#000000' : '#FFFFFF');
      }
    });

    //붙여넣기 처리
    document.addEventListener('paste', (event: any) => {
      const items = (event.clipboardData || event.originalEvent.clipboardData).items;

      for (let i = 0; i < items.length; i++) {
        if (items[i].type.indexOf('image') !== -1) {
          const blob = items[i].getAsFile();

          
          img.src = URL.createObjectURL(blob);

          img.onload = function() {
            drawImage(ctx, canvas.value, img);
          }
        }
      }      
    });
  }
  //이미지 그리기 함수
  function drawImage(ctx: any, canvas: any, img: any) {
    const aspectRatio = img ? img.width / img.height : 1;
    const canvasAspectRatio = canvas.width / canvas.height;

    let imgWidth = canvas.width;
    let imgHeight = canvas.height;

    if (aspectRatio > canvasAspectRatio) {
      imgWidth = canvas.width;
      imgHeight = imgWidth / aspectRatio;
    } else {
      imgHeight = canvas.height;
      imgWidth = imgHeight * aspectRatio;
    }

    const x = (canvas.width - imgWidth) / 2;
    const y = (canvas.height - imgHeight) / 2;

    ctx.clearRect(0, 0, canvas.width, canvas.height);

    ctx.drawImage(img, x, y, imgWidth, imgHeight);

  }
});
</script>

<style scoped>
.container{
  flex: 1;
  display: flex;
}
  .myDiv{
    flex:1;

    min-height: 100vh;
    display: flex;
    align-items: center;
  }
  .palette {
    width: 200px;
    height: 100vh;
    background-color: #f2f2f2;
    overflow: auto;
  }
  .item {
    width: 100%;
    height: 8rem;
    display: flex;
    justify-content: center;
    align-items: center;
  }
  .title {
    position: fixed;
    margin: 5px;
  }
</style>