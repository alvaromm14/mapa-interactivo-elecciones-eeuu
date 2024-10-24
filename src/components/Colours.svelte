<script>
    function countColorsNebraska() {
        const balls = document.querySelectorAll(
            "#nebraska .bola, #nebraska .bola-grande",
        );
        const colorCount = { "#C84441": 0, "#148ECC": 0, "#B3B3B3": 0 };

        balls.forEach((ball) => {
            const bgColor = ball.style.backgroundColor;
            if (bgColor === "rgb(200, 68, 65)") colorCount["#C84441"]++;
            else if (bgColor === "rgb(20, 142, 204)") colorCount["#148ECC"]++;
            else if (bgColor === "rgb(179, 179, 179)") colorCount["#B3B3B3"]++;
        });

        return colorCount;
    }

    function countColorsMaine() {
        const balls = document.querySelectorAll(
            "#maine .bola, #maine .bola-grande",
        );
        const colorCount = { "#C84441": 0, "#148ECC": 0, "#B3B3B3": 0 };

        balls.forEach((ball) => {
            const bgColor = ball.style.backgroundColor;
            if (bgColor === "rgb(200, 68, 65)") colorCount["#C84441"]++;
            else if (bgColor === "rgb(20, 142, 204)") colorCount["#148ECC"]++;
            else if (bgColor === "rgb(179, 179, 179)") colorCount["#B3B3B3"]++;
        });

        return colorCount;
    }

    function getDominantColor(colorCount) {
        return Object.keys(colorCount).reduce((a, b) =>
            colorCount[a] > colorCount[b] ? a : b,
        );
    }

    // Define una función para obtener el color de Nebraska
    function getNebraskaFillColor() {
        const colorCount = countColorsNebraska();
        return getDominantColor(colorCount);
    }

    // Define una función para obtener el color de Maine
    function getMaineFillColor() {
        const colorCount = countColorsMaine();
        return getDominantColor(colorCount);
    }
</script>
