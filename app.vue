<template>
  <div>
    <div>Snake {{ status === Status.Done ? 'LOSE' : '' }}</div>
    <Canvas>
      <!-- <Grid v-for="(g, i) in Array(size * size)" :key="i">{{ i }}</Grid> -->
      <Grid v-for="{ x, y } in snake" :key="x + '-' + y" :pos="{ x, y }" />
      <Grid v-if="food" food :pos="food" />
    </Canvas>
  </div>
</template>

<script>
const Direction = {
  Up: 'Up',
  Down: 'Down',
  Left: 'Left',
  Right: 'Right',
};
const Status = {
  Play: 'Play',
  Pause: 'Pause',
  Done: 'Done',
};
const getDirection = (newDir, oldDir) => {
  switch (oldDir) {
    case Direction.Up:
      return newDir === Direction.Down ? oldDir : newDir;
    case Direction.Down:
      return newDir === Direction.Up ? oldDir : newDir;
    case Direction.Left:
      return newDir === Direction.Right ? oldDir : newDir;
    case Direction.Right:
      return newDir === Direction.Left ? oldDir : newDir;
    default:
      return oldDir;
  }
};
const getStep = (direction, x, y, backward = false) => {
  const adder = backward ? -1 : 1;
  switch (direction) {
    case Direction.Up:
      return { x: x, y: y - adder };
    case Direction.Down:
      return { x: x, y: y + adder };
    case Direction.Left:
      return { x: x - adder, y: y };
    case Direction.Right:
      return { x: x + adder, y: y };
    default:
      return { x, y };
  }
};
const getClippedStep = (direction, oriX, oriY, max, backward) => {
  const { x, y } = getStep(direction, oriX, oriY, backward);
  return { x: x % max || max, y: y % max || max };
};
const getRandomFood = (snake, max) => {
  const randomFood = Math.ceil(Math.random() * max);
  if (snake.some(({ x, y }) => x === randomFood && y === randomFood))
    return getRandomFood(snake, max);
  return randomFood;
};
</script>

<script setup>
const size = ref(12);
const direction = ref(Direction.Right);
const snake = ref([]);
const food = ref(null);
const status = ref(Status.Pause);
const interval = ref(null);
const speed = ref(450);

onMounted(() => {
  document.addEventListener('keyup', handleKeyUp);
});
onUnmounted(() => {
  document.removeEventListener('keyup', handleKeyUp);
  clearInterval(interval.value);
});

const handleKeyUp = (e) => {
  switch (e.code) {
    case 'Space':
      return togglePlay();
    case 'Enter':
      return resetGame();
    case 'ArrowUp':
    case 'ArrowDown':
    case 'ArrowLeft':
    case 'ArrowRight': {
      direction.value = getDirection(
        Direction[e.code.replace('Arrow', '')],
        direction.value
      );
      return;
    }
    default:
      return;
  }
};
const togglePlay = () => {
  if (status.value === Status.Play) {
    status.value = Status.Pause;
    clearInterval(interval.value);
    interval.value = null;
    return;
  }
  if (status.value === Status.Done || !food.value) return resetGame();
  status.value = Status.Play;
  interval.value = setInterval(updateSnake, speed.value);
};
const resetGame = () => {
  // Stop game
  status.value = Status.Pause;
  clearInterval(interval.value);
  interval.value = null;

  const randomSnake = Math.ceil(Math.random() * size.value);
  const randomDirection = Math.floor(
    Math.random() * Object.keys(Direction).length
  );

  direction.value = Direction[Object.keys(Direction)[randomDirection]];
  snake.value = [
    { x: randomSnake, y: randomSnake },
    getClippedStep(direction.value, randomSnake, randomSnake, size.value, true),
  ];

  const randomFood = getRandomFood(snake.value, size.value);
  food.value = { x: randomFood, y: randomFood };

  // Start game
  status.value = Status.Play;
  interval.value = setInterval(updateSnake, speed.value);
};
const updateSnake = () => {
  const head = getClippedStep(
    direction.value,
    snake.value[0].x,
    snake.value[0].y,
    size.value
  );
  console.log(direction.value, head, snake.value, food.value);

  // Eat food
  if (head.x === food.value.x && head.y === food.value.y) {
    // Elongate body
    snake.value = [head, ...snake.value];
    const randomFood = getRandomFood(snake.value, size.value);
    food.value = { x: randomFood, y: randomFood };
    return;
  }

  // Eat self
  if (snake.value.some(({ x, y }) => x === head.x && y === head.y)) {
    // Pause the game and set status to Done
    togglePlay();
    status.value = Status.Done;
    return;
  }

  // Move
  snake.value = [head, ...snake.value.slice(0, -1)];
};
</script>
