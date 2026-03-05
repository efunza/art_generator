import io
import math
import numpy as np
import matplotlib.pyplot as plt
import streamlit as st

st.set_page_config(page_title="Mathematical Art Generator", page_icon="🎨", layout="wide")

# ----------------------------
# Helpers
# ----------------------------
def fig_to_png_bytes(fig) -> bytes:
    buf = io.BytesIO()
    fig.savefig(buf, format="png", dpi=220, bbox_inches="tight", pad_inches=0.05)
    plt.close(fig)
    buf.seek(0)
    return buf.getvalue()

def setup_ax(figsize=(7, 7), bg="#0b0f14"):
    fig, ax = plt.subplots(figsize=figsize)
    fig.patch.set_facecolor(bg)
    ax.set_facecolor(bg)
    ax.set_aspect("equal", "box")
    ax.axis("off")
    return fig, ax

def normalize_axes(ax, pad=1.05):
    ax.relim()
    ax.autoscale_view()
    x0, x1 = ax.get_xlim()
    y0, y1 = ax.get_ylim()
    cx, cy = (x0 + x1) / 2, (y0 + y1) / 2
    rx, ry = (x1 - x0) / 2, (y1 - y0) / 2
    r = max(rx, ry) * pad + 1e-9
    ax.set_xlim(cx - r, cx + r)
    ax.set_ylim(cy - r, cy + r)

# ----------------------------
# Pattern generators (Matplotlib)
# ----------------------------
def draw_spirograph(ax, R=9.0, r=4.0, d=7.0, turns=8, points=4000, linewidth=1.2):
    # Hypotrochoid: x=(R-r)cos(t)+d cos((R-r)/r * t), y=(R-r)sin(t)-d sin((R-r)/r * t)
    t = np.linspace(0, 2 * np.pi * turns, points)
    k = (R - r) / r
    x = (R - r) * np.cos(t) + d * np.cos(k * t)
    y = (R - r) * np.sin(t) - d * np.sin(k * t)
    ax.plot(x, y, linewidth=linewidth)

def draw_mandala(ax, petals=8, iterations=80, base_radius=10.0, ring_step=0.13, jitter=0.0, linewidth=1.0):
    # Radial symmetric rosette layers
    for i in range(iterations):
        r = base_radius * (1 - i * ring_step / max(1, iterations))
        if r <= 0:
            break
        theta = np.linspace(0, 2 * np.pi, 900)
        # rose curve variation: r(θ) = r * (1 + 0.25 cos(petals θ))
        rr = r * (1 + 0.25 * np.cos(petals * theta))
        if jitter > 0:
            rr = rr * (1 + jitter * np.sin((i + 1) * theta))
        x = rr * np.cos(theta)
        y = rr * np.sin(theta)
        ax.plot(x, y, linewidth=linewidth)

def draw_tessellation(ax, rows=12, cols=12, cell=1.0, skew=0.0, rotation_deg=0.0, linewidth=0.8):
    # Simple geometric tessellation: rotated + skewed square grid with diagonal motifs
    rot = math.radians(rotation_deg)
    c, s = math.cos(rot), math.sin(rot)

    def transform(x, y):
        # shear in x by skew*y then rotate
        x2 = x + skew * y
        y2 = y
        return (c * x2 - s * y2, s * x2 + c * y2)

    for r in range(rows):
        for col in range(cols):
            x0 = (col - cols / 2) * cell
            y0 = (r - rows / 2) * cell
            # square corners
            pts = [(x0, y0), (x0 + cell, y0), (x0 + cell, y0 + cell), (x0, y0 + cell), (x0, y0)]
            tx, ty = zip(*[transform(x, y) for x, y in pts])
            ax.plot(tx, ty, linewidth=linewidth)

            # diagonal motif
            a = transform(x0, y0)
            b = transform(x0 + cell, y0 + cell)
            c1 = transform(x0 + cell, y0)
            d1 = transform(x0, y0 + cell)
            ax.plot([a[0], b[0]], [a[1], b[1]], linewidth=linewidth * 0.9)
            ax.plot([c1[0], d1[0]], [c1[1], d1[1]], linewidth=linewidth * 0.9)

def draw_radial_star(ax, points=10, iterations=100, inner=3.0, outer=10.0, twist_deg=0.0, linewidth=1.1):
    # Iterated star polygons with twist
    twist = math.radians(twist_deg)
    for i in range(iterations):
        t = i / max(1, iterations - 1)
        R = outer * (1 - 0.7 * t)
        r = inner * (1 + 0.3 * t)

        angles = np.linspace(0, 2 * np.pi, points * 2 + 1)[:-1]
        radii = np.array([R if k % 2 == 0 else r for k in range(points * 2)])
        ang = angles + twist * i / max(1, iterations)
        x = radii * np.cos(ang)
        y = radii * np.sin(ang)
        x = np.append(x, x[0])
        y = np.append(y, y[0])
        ax.plot(x, y, linewidth=linewidth)

# ----------------------------
# UI
# ----------------------------
st.title("🎨 Mathematical Art Generator (Geometry + Iteration)")
st.caption(
    "Generates Spirograph, Mandala, Tessellation, and Radial Star patterns using symmetry, rotation, translation-style grids, and iteration."
)

with st.sidebar:
    st.header("Controls")
    pattern = st.selectbox("Pattern Type", ["Spirograph", "Mandala", "Tessellation", "Radial Star"])
    st.divider()

    # Global style
    bg = st.color_picker("Background", "#0b0f14")
    linewidth = st.slider("Line Width", 0.2, 3.0, 1.1, 0.1)

    st.divider()
    st.subheader("Pattern Parameters")

    params = {}

    if pattern == "Spirograph":
        params["R"] = st.slider("Big radius (R)", 2.0, 20.0, 9.0, 0.5)
        params["r"] = st.slider("Small radius (r)", 1.0, 15.0, 4.0, 0.5)
        params["d"] = st.slider("Pen distance (d)", 0.0, 20.0, 7.0, 0.5)
        params["turns"] = st.slider("Turns (iteration)", 1, 30, 8, 1)
        params["points"] = st.slider("Smoothness (points)", 600, 12000, 4000, 200)

    elif pattern == "Mandala":
        params["petals"] = st.slider("Symmetry axes / petals", 3, 24, 8, 1)
        params["iterations"] = st.slider("Iterations (rings)", 5, 250, 80, 5)
        params["base_radius"] = st.slider("Base radius", 2.0, 20.0, 10.0, 0.5)
        params["ring_step"] = st.slider("Ring shrink step", 0.01, 0.5, 0.13, 0.01)
        params["jitter"] = st.slider("Wave variation", 0.0, 0.4, 0.0, 0.01)

    elif pattern == "Tessellation":
        params["rows"] = st.slider("Rows (iteration)", 3, 40, 12, 1)
        params["cols"] = st.slider("Cols (iteration)", 3, 40, 12, 1)
        params["cell"] = st.slider("Cell size", 0.4, 3.0, 1.0, 0.1)
        params["skew"] = st.slider("Skew (shear)", -1.0, 1.0, 0.0, 0.05)
        params["rotation_deg"] = st.slider("Rotation (degrees)", 0.0, 90.0, 0.0, 1.0)

    elif pattern == "Radial Star":
        params["points"] = st.slider("Star points (symmetry axes)", 3, 30, 10, 1)
        params["iterations"] = st.slider("Iterations", 5, 250, 100, 5)
        params["inner"] = st.slider("Inner radius", 0.5, 15.0, 3.0, 0.5)
        params["outer"] = st.slider("Outer radius", 2.0, 30.0, 10.0, 0.5)
        params["twist_deg"] = st.slider("Twist per iteration (deg)", 0.0, 30.0, 0.0, 0.5)

    st.divider()
    seed = st.number_input("Seed (optional)", min_value=0, max_value=10_000_000, value=0, step=1)
    randomize = st.button("🎲 Randomize")

# Randomize (lightweight)
if randomize:
    rng = np.random.default_rng(int(seed) if seed else None)
    if pattern == "Spirograph":
        params["R"] = float(rng.uniform(5, 16))
        params["r"] = float(rng.uniform(1, 10))
        params["d"] = float(rng.uniform(0, 14))
        params["turns"] = int(rng.integers(3, 16))
        params["points"] = int(rng.integers(2000, 8000))
    elif pattern == "Mandala":
        params["petals"] = int(rng.integers(5, 18))
        params["iterations"] = int(rng.integers(40, 160))
        params["base_radius"] = float(rng.uniform(6, 14))
        params["ring_step"] = float(rng.uniform(0.05, 0.22))
        params["jitter"] = float(rng.uniform(0.0, 0.25))
    elif pattern == "Tessellation":
        params["rows"] = int(rng.integers(8, 22))
        params["cols"] = int(rng.integers(8, 22))
        params["cell"] = float(rng.uniform(0.6, 1.8))
        params["skew"] = float(rng.uniform(-0.5, 0.5))
        params["rotation_deg"] = float(rng.uniform(0, 60))
    else:
        params["points"] = int(rng.integers(6, 18))
        params["iterations"] = int(rng.integers(50, 160))
        params["inner"] = float(rng.uniform(1.5, 6))
        params["outer"] = float(rng.uniform(8, 18))
        params["twist_deg"] = float(rng.uniform(0, 8))

# Render
colA, colB = st.columns([1.25, 1], gap="large")

with colA:
    fig, ax = setup_ax(bg=bg)

    if pattern == "Spirograph":
        draw_spirograph(ax, linewidth=linewidth, **params)
    elif pattern == "Mandala":
        draw_mandala(ax, linewidth=linewidth, **params)
    elif pattern == "Tessellation":
        draw_tessellation(ax, linewidth=linewidth, **params)
    else:
        draw_radial_star(ax, linewidth=linewidth, **params)

    normalize_axes(ax, pad=1.06)
    st.pyplot(fig, use_container_width=True)

    png_bytes = fig_to_png_bytes(fig)
    st.download_button(
        "⬇️ Download as PNG",
        data=png_bytes,
        file_name=f"math_art_{pattern.replace(' ', '_').lower()}.png",
        mime="image/png",
        use_container_width=True,
    )

with colB:
    st.subheader("What you’re controlling")
    if pattern == "Spirograph":
        st.markdown(
            "- **R, r, d** change the curve shape\n"
            "- **Turns** controls how long the curve iterates\n"
            "- **Smoothness** controls how many points are plotted"
        )
    elif pattern == "Mandala":
        st.markdown(
            "- **Petals** ≈ symmetry axes\n"
            "- **Iterations** = number of rings\n"
            "- **Ring step** controls how fast rings shrink\n"
            "- **Wave variation** adds extra geometry variation"
        )
    elif pattern == "Tessellation":
        st.markdown(
            "- **Rows/Cols** set repetition (iteration)\n"
            "- **Skew** shears the grid (a geometric transformation)\n"
            "- **Rotation** rotates the whole tiling"
        )
    else:
        st.markdown(
            "- **Star points** ≈ symmetry axes\n"
            "- **Iterations** stacks many stars for complexity\n"
            "- **Twist** rotates each iteration a bit"
        )

    st.divider()
    st.subheader("Project alignment")
    st.markdown(
        "This app matches your report’s idea of generating art through **symmetry, rotation, translation-style repetition, and iteration**, "
        "and supports the four listed patterns (spirograph, mandala, tessellation, radial designs)."
    )
    st.caption("Next step idea: add animation (as you recommended) by rendering frames and playing them in Streamlit.")  #  below

st.caption("Report recommendation: add animation + GUI features. :contentReference[oaicite:4]{index=4}")
