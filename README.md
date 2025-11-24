<!DOCTYPE html>
<html>
<head>
<style>
body{
    margin:0;
    height:100vh;
    background: linear-gradient(135deg, #4b79a1, #283e51);
    display:flex;
    justify-content:center;
    align-items:center;
    font-family:Arial;
    color:white;
    font-size:40px;
}
</style>
</head>
<body>
   Beautiful Gradient Background
</body>
</html>
<!DOCTYPE html>
<html>
<head>
<style>
body {
    margin: 0;
    height: 100vh;
    background: linear-gradient(-45deg, #4facfe, #00f2fe, #43e97b, #fa709a);
    background-size: 400% 400%;
    animation: gradientAnim 10s ease infinite;
    display:flex;
    justify-content:center;
    align-items:center;
    font-family: Arial;
    color: white;
    font-size: 40px;
}
@keyframes gradientAnim {
    0% { background-position: 0% 50%; }
    50% { background-position: 100% 50%; }
    100% { background-position: 0% 50%; }
}
</style>
</head>
<body>
   Animated Gradient Background
</body>
</html>
<!DOCTYPE html>
<html>
<head>
<style>
body {
    margin: 0;
    overflow: hidden;
    background: #0f0f0f;
    font-family: Arial;
    color: white;
}
#text {
    position: absolute;
    top: 40%;
    width: 100%;
    text-align: center;
    font-size: 40px;
}
</style>
</head>
<body>

<canvas id="bg"></canvas>
<div id="text">Particle Background</div>

<script>
const canvas = document.getElementById("bg");
const ctx = canvas.getContext("2d");

canvas.width = window.innerWidth;
canvas.height = window.innerHeight;

let particles = [];

for (let i = 0; i < 120; i++) {
    particles.push({
        x: Math.random() * canvas.width,
        y: Math.random() * canvas.height,
        size: Math.random() * 3,
        speedX: (Math.random() - 0.5) * 1.2,
        speedY: (Math.random() - 0.5) * 1.2
    });
}

function animate() {
    ctx.fillStyle = "rgba(0,0,0,0.20)";
    ctx.fillRect(0, 0, canvas.width, canvas.height);

    particles.forEach(p => {
        ctx.fillStyle = "#00eaff";
        ctx.beginPath();
        ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2);
        ctx.fill();

        p.x += p.speedX;
        p.y += p.speedY;

        if (p.x < 0 || p.x > canvas.width) p.speedX *= -1;
        if (p.y < 0 || p.y > canvas.height) p.speedY *= -1;
    });

    requestAnimationFrame(animate);
}

animate();
</script>

</body>
</html>
