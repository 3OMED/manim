from manim import *
import numpy as np


class GreenTheorem(Scene):
    def construct(self):
        # ---------- 1) Title ----------
        title = Tex(r"Green's Theorem", font_size=64, color=YELLOW)
        self.play(Write(title))
        self.wait(0.5)
        self.play(title.animate.scale(0.6).to_edge(UP))

        # ---------- 2) Statement of the theorem ----------
        theorem = MathTex(
            r"\oint_C L\,dx + M\,dy",
            r"=",
            r"\iint_D \left( \frac{\partial M}{\partial x} - \frac{\partial L}{\partial y} \right) dA",
            font_size=44,
        )
        theorem[0].set_color(BLUE)
        theorem[2].set_color(GREEN)
        self.play(Write(theorem))
        self.wait(1)

        left_label = Tex("Line integral around the boundary", font_size=28, color=BLUE)
        right_label = Tex("Double integral over the region", font_size=28, color=GREEN)
        left_label.next_to(theorem[0], DOWN, buff=0.6)
        right_label.next_to(theorem[2], DOWN, buff=0.6)
        self.play(FadeIn(left_label, shift=UP * 0.3), FadeIn(right_label, shift=UP * 0.3))
        self.wait(1.5)

        self.play(FadeOut(left_label), FadeOut(right_label))
        self.play(theorem.animate.scale(0.6).next_to(title, DOWN, buff=0.3))

        # ---------- 3) Geometry: region D, boundary C, vector field ----------
        R = 2
        plane = NumberPlane(
            x_range=[-3, 3, 1], y_range=[-3, 3, 1],
            x_length=5.4, y_length=5.4,
            background_line_style={"stroke_opacity": 0.25},
        )
        # Use scene coordinates directly: 1 unit = plane unit * 0.9
        unit = 5.4 / 6
        region = Circle(radius=R * unit, color=GREEN, fill_opacity=0.25, stroke_width=0)
        boundary = Circle(radius=R * unit, color=BLUE, stroke_width=5)

        field = ArrowVectorField(
            lambda p: np.array([-p[1], p[0], 0]),
            x_range=[-3, 3, 1], y_range=[-3, 3, 1],
            length_func=lambda norm: 0.45 * sigmoid(norm),
            colors=[TEAL, YELLOW, RED],
        )
        field.scale(unit)  # only used as a visual aid

        picture = VGroup(plane, field, region, boundary)
        picture.scale(0.85).to_edge(LEFT, buff=0.6).shift(DOWN * 0.7)
        # Re-reference the circles after scaling/moving
        center = picture.get_center()
        circle_path = Circle(radius=R * unit * 0.85, color=BLUE).move_to(center)

        self.play(Create(plane), run_time=1)
        self.play(FadeIn(field), run_time=1)
        self.play(FadeIn(region))
        self.play(Create(boundary))

        D_label = MathTex("D", color=GREEN, font_size=40).move_to(center)
        C_label = MathTex("C", color=BLUE, font_size=40).move_to(
            center + (R * unit * 0.85 + 0.35) * np.array([np.cos(PI / 4), np.sin(PI / 4), 0])
        )
        self.play(Write(D_label), Write(C_label))

        # Counter-clockwise traversal of C
        dot = Dot(color=RED).move_to(circle_path.point_from_proportion(0))
        trace = TracedPath(dot.get_center, stroke_color=RED, stroke_width=6)
        self.add(trace)
        self.play(FadeIn(dot))
        self.play(MoveAlongPath(dot, circle_path), run_time=4, rate_func=linear)
        ccw = Tex("Counter-clockwise orientation", font_size=26, color=RED)
        ccw.next_to(picture, DOWN, buff=0.15)
        self.play(FadeIn(ccw))
        self.play(FadeOut(dot), FadeOut(trace))
        self.wait(0.5)

        # ---------- 4) Worked example ----------
        example_title = Tex(r"Example: $L=-y,\; M=x$, \; $D$: circle of radius 2", font_size=32)
        example_title.next_to(theorem, DOWN, buff=0.4).to_edge(RIGHT, buff=0.5)

        lhs_title = Tex("Left side (line integral)", font_size=28, color=BLUE)
        lhs1 = MathTex(r"x=2\cos t,\; y=2\sin t,\; 0\le t\le 2\pi", font_size=30)
        lhs2 = MathTex(
            r"\oint_C -y\,dx + x\,dy = \int_0^{2\pi}"
            r"\left(4\sin^2 t + 4\cos^2 t\right) dt",
            font_size=30,
        )
        lhs3 = MathTex(r"= \int_0^{2\pi} 4\,dt = 8\pi", font_size=30, color=BLUE)

        rhs_title = Tex("Right side (double integral)", font_size=28, color=GREEN)
        rhs1 = MathTex(
            r"\frac{\partial M}{\partial x} - \frac{\partial L}{\partial y} = 1-(-1) = 2",
            font_size=30,
        )
        rhs2 = MathTex(
            r"\iint_D 2\,dA = 2\cdot \pi (2)^2 = 8\pi",
            font_size=30, color=GREEN,
        )

        block = VGroup(
            example_title,
            lhs_title, lhs1, lhs2, lhs3,
            rhs_title, rhs1, rhs2,
        ).arrange(DOWN, aligned_edge=LEFT, buff=0.18)
        block.next_to(picture, RIGHT, buff=0.5)
        # keep inside the frame
        if block.width > config.frame_width - picture.width - 1.4:
            block.scale_to_fit_width(config.frame_width - picture.width - 1.4)
        block.align_to(picture, UP).shift(UP * 0.2)

        self.play(Write(example_title))
        self.play(Write(lhs_title), Write(lhs1))
        self.play(Write(lhs2))
        self.play(Write(lhs3))
        self.wait(0.5)
        self.play(Write(rhs_title), Write(rhs1))
        self.play(Write(rhs2))
        self.wait(0.5)

        # ---------- 5) Conclusion ----------
        box = SurroundingRectangle(VGroup(lhs3, rhs2), color=YELLOW, buff=0.15)
        verdict = MathTex(r"8\pi = 8\pi \;\checkmark", color=YELLOW, font_size=42)
        verdict.next_to(box, DOWN, buff=0.3)
        self.play(Create(box), Write(verdict))
        self.wait(3)
