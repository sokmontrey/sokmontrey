<script>
	import {
		draw_to_svg,
		diagram_combine,
		V2,
		Interactive,
		arrow1,
		text,
		textvar,
		xygrid,
	} from "diagramatics";

	import { interpolate } from "d3-interpolate";

	import { plotf } from "diagramatics";
	import { onMount } from "svelte";
	import { getRootColor } from "@/utils/color";

	let mysvg;
	let controldiv;

	let draw = (...diagrams) => {
		draw_to_svg(mysvg, diagram_combine(...diagrams));
	};
	const f = (x) => x ** 2;
	const df = (x) => 2 * x;
	const slope = (a, b) => (b.y - a.y) / (b.x - a.x);
	// const axis_opt = { x: [-5, 5], y: [1, 5] };
	const axis_opt = { xrange: [-1.5, 1.5], yrange: [-1, 2] };
	const f_p = plotf(f, axis_opt).stroke("var(--nt-color").strokewidth(2);
	// const axis_p = axes_empty(axis_opt);
	const axis_p = xygrid(axis_opt);

	onMount(() => {
		const b_color = interpolate(
			getRootColor("--pm-color"),
			getRootColor("--sd-color"),
		);

		let int = new Interactive(controldiv, mysvg);

		int.draw_function = (inp) => {
			const a = inp.a;
			const b = inp.b;

			const der_rate = df(a.x);
			const avg_rate = slope(a, b);

			const a_arrow = arrow1(a, a.add(V2(0.5, der_rate / 2)), 0.12)
				.fill("blue")
				.stroke("blue")
				.opacity(0.5);

			const t = Math.abs(a.x - b.x) ** 2 / 2.8;
			const color = b_color(t);
			const b_arrow = arrow1(
				a,
				a.add(V2(0.5, avg_rate / 2).scale(b.x > a.x ? 1 : -1)),
				0.12,
			)
				.fill(color)
				.stroke(color);

			const a_text = text("a")
				.position(a.add(V2(0, -0.5)))
				.fontsize(6)
				.textfill("var(--pm-color)");

			const b_text = text("b")
				.position(b.add(V2(0, -0.5)))
				.fontsize(6)
				.textfill("var(--sd-color)");

			draw(f_p, axis_p, a_arrow, b_arrow, a_text, b_text);
		};

		int.locator("a", V2(-1, 1), 0.12, "blue", f_p);
		int.locator("b", V2(1, 1), 0.12, "red", f_p);
		int.draw();
		int.locator_initial_draw();
	});
</script>

<div class="flex justify-center items-center">
	<svg bind:this={mysvg} viewbox="0 0 100 100" width="100%" height="100%"></svg>
	<div bind:this={controldiv}></div>
</div>
