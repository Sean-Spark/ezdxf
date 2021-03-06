.. module:: ezdxf.path

Path
====

This module implements a geometric :class:`Path`, supported by several render
backends, with the goal to create such paths from DXF entities like LWPOLYLINE,
POLYLINE or HATCH and send them to the render backend,
see :mod:`ezdxf.addons.drawing`.

Minimum common interface:

- matplotlib: `PathPatch`_
    - matplotlib.path.Path() codes:
    - MOVETO
    - LINETO
    - CURVE3 - quadratic Bèzier-curve
    - CURVE4 - cubic Bèzier-curve

- PyQt: `QPainterPath`_
    - moveTo()
    - lineTo()
    - quadTo() - quadratic Bèzier-curve (converted to a cubic Bèzier-curve)
    - cubicTo() - cubic Bèzier-curve

- PyCairo: `Context`_
    - move_to()
    - line_to()
    - no support for quadratic Bèzier-curve
    - curve_to() - cubic Bèzier-curve

- SVG: `SVG-Path`_
    - "M" - absolute move to
    - "L" - absolute line to
    - "Q" - absolute quadratic Bèzier-curve
    - "C" - absolute cubic Bèzier-curve

ARC and ELLIPSE entities are approximated by multiple cubic Bézier-curves, which
are close enough for display rendering. Non-rational SPLINES of 3rd degree can
be represented exact as multiple cubic Bézier-curves, other B-splines will be
approximated. The XLINE and the RAY entities are not supported, because of their
infinite nature.

This :class:`Path` class is a full featured 3D object, although the backends
only support 2D paths.

.. hint::

    A :class:`Path` can not represent a point. A :class:`Path` with only a
    start point yields no vertices!

.. versionchanged:: 0.16
    Refactored the module :mod:`ezdxf.render.path` into the subpackage
    :mod:`ezdxf.path`.

The usability of the :class:`Path` class expanded by the introduction
of the the revers conversion from :class:`Path` to DXF entities (LWPOLYLINE,
POLYLINE, LINE) and many other tools in `ezdxf` v0.16.
To emphasize this new usability, the :class:`Path` class has got its own
subpackage :mod:`ezdxf.path`.

.. warning::

    Always import from the top level :mod:`ezdxf.path`, never from the
    sub-modules

Factory Functions
-----------------

Functions to create :class:`Path` objects from other objects.

.. function:: make_path(entity: DXFEntity) -> Path

    Factory function to create a single :class:`Path` object from a DXF
    entity. Supported DXF types:

    - LINE
    - CIRCLE
    - ARC
    - ELLIPSE
    - SPLINE and HELIX
    - LWPOLYLINE
    - 2D and 3D POLYLINE
    - SOLID, TRACE, 3DFACE
    - IMAGE, WIPEOUT clipping path
    - VIEWPORT clipping path

    The HATCH entity consist of multiple boundary paths and is not convertible
    into a single :class:`Path` object and therefore not supported by this
    function.

    :param entity: DXF entity
    :param segments: minimal count of cubic Bézier-curves for elliptical arcs
        like CIRCLE, ARC, ELLIPSE, see :meth:`Path.add_ellipse`
    :param level: subdivide level for SPLINE approximation,
        see :meth:`Path.add_spline`

    :raises TypeError: for unsupported DXF types

    .. versionadded:: 0.16


.. autofunction:: from_hatch(hatch: Hatch) -> Iterable[Path]

.. autofunction:: from_vertices(vertices: Iterable[Vertex], close=False) -> Path

.. autofunction:: from_matplotlib_path(mpath, curves=True) -> Iterable[Path]

.. autofunction:: from_qpainter_path(qpath) -> Iterable[Path]

Render Functions
----------------

Functions to create DXF entities from paths and add them to the modelspace, a
paperspace layout or a block definition.

.. autofunction:: render_lwpolylines(layout: Layout, paths: Iterable[Path], *, distance: float = 0.01, segments: int = 4, extrusion: Vertex = (0, 0, 1),  dxfattribs: Dict = None) -> EntityQuery

.. autofunction:: render_polylines2d(layout: Layout, paths: Iterable[Path], *, distance: float = 0.01, segments: int = 4, extrusion: Vertex = (0, 0, 1),  dxfattribs: Dict = None) -> EntityQuery

.. autofunction:: render_hatches(layout: Layout, paths: Iterable[Path], *, edge_path = True, distance: float = 0.01, segments: int = 4, g1_tol: float = 1e-4, extrusion: Vertex = (0, 0, 1),  dxfattribs: Dict = None) -> EntityQuery

.. autofunction:: render_polylines3d(layout: Layout, paths: Iterable[Path], *, distance: float = 0.01, segments: int = 4, dxfattribs: Dict = None) -> EntityQuery

.. autofunction:: render_lines(layout: Layout, paths: Iterable[Path], *, distance: float = 0.01, segments: int = 4, dxfattribs: Dict = None) -> EntityQuery

.. autofunction:: render_splines_and_polylines(layout: Layout, paths: Iterable[Path], *, g1_tol: float = 1e-4, dxfattribs: Dict = None) -> EntityQuery

Entity Maker
------------

Functions to create DXF entities from paths.

.. autofunction:: to_lwpolylines(paths: Iterable[Path], *, distance: float = 0.01, segments: int = 4, extrusion: Vertex = (0, 0, 1),  dxfattribs: Dict = None) -> Iterable[LWPolyline]

.. autofunction:: to_polylines2d(paths: Iterable[Path], *, distance: float = 0.01, segments: int = 4, extrusion: Vertex = (0, 0, 1),  dxfattribs: Dict = None) -> Iterable[Polyline]

.. autofunction:: to_hatches(paths: Iterable[Path], *, edge_path: True, distance: float = 0.01, segments: int = 4, g1_tol: float = 1e-4, extrusion: Vertex = (0, 0, 1),  dxfattribs: Dict = None) -> Iterable[Hatch]

.. autofunction:: to_polylines3d(paths: Iterable[Path], *, distance: float = 0.01, segments: int = 4, dxfattribs: Dict = None) -> Iterable[Polyline]

.. autofunction:: to_lines(paths: Iterable[Path], *, distance: float = 0.01, segments: int = 4, dxfattribs: Dict = None) -> Iterable[Line]

.. autofunction:: to_splines_and_polylines(paths: Iterable[Path], *, g1_tol: float= 1e-4, dxfattribs: Dict = None) -> Iterable[Union[Spline, Polyline]]

Tool Maker
----------

Functions to create construction tools.

.. autofunction:: to_bsplines_and_vertices(path: Path, g1_tol: float = 1e-4) -> Iterable[Union[BSpline, List[Vec3]]]

.. autofunction:: to_matplotlib_path(paths: Iterable[Path], extrusion = (0, 0, 1)) -> matplotlib.path.Path

.. autofunction:: to_qpainter_path(paths: Iterable[Path], extrusion = (0, 0, 1)) -> PyQt5.QtGui.QPainterPath


Utility Functions
-----------------

.. autofunction:: transform_paths(paths: Iterable[Path], m: Matrix44) -> List[Path]

.. autofunction:: transform_paths_to_ocs(paths: Iterable[Path], ocs: OCS) -> List[Path]

.. autofunction:: bbox(paths: Iterable[Path]) -> BoundingBox

.. autofunction:: fit_paths_into_box(paths: Iterable[Path], size: Tuple[float, float, float], uniform = True, source_box: BoundingBox = None) -> List[Path]

.. autofunction:: add_bezier3p(path: Path, curves: Iterable[Bezier3P])

.. autofunction:: add_bezier4p(path: Path, curves: Iterable[Bezier4P])

.. autofunction:: add_ellipse(path: Path,ellipse: ConstructionEllipse, segments=1)

.. autofunction:: add_spline(path: Path, spline: BSpline, level=4)

Basic Shapes
------------

.. autofunction:: unit_circle(start_angle: float = 0, end_angle: float = 2π, segments: int = 1, transform: Matrix44 = None) -> Path

.. autofunction:: wedge(start_angle: float, end_angle: float, segments: int = 1, transform: Matrix44 = None) -> Path

.. autofunction:: elliptic_transformation(center: Vertex = (0, 0, 0), radius: float = 1, ratio: float = 1, rotation: float = 0) -> Matrix44

.. autofunction:: rect(width: float = 1, height: float = 1, transform: Matrix44 = None) -> Path

.. autofunction:: ngon(count: int, length: float = None, radius: float = 1.0, transform: Matrix44 = None) -> Path

.. autofunction:: star(count: int, r1: float, r2: float, transform: Matrix44 = None) -> Path

.. autofunction:: gear(count: int, top_width: float, bottom_width: float, height: float, outside_radius: float, transform: Matrix44 = None) -> Path

The :mod:`~ezdxf.addons.text2path` add-on provides additional functions to
create paths from text strings and DXF text entities.


The Path Class
--------------

.. class:: Path

    .. autoattribute:: start

    .. autoattribute:: end

    .. autoattribute:: is_closed

    .. autoattribute:: has_lines

    .. autoattribute:: has_curves

    .. automethod:: control_vertices

    .. automethod:: has_clockwise_orientation

    .. automethod:: line_to(location: Vec3)

    .. automethod:: curve3_to(location: Vec3, ctrl: Vec3)

    .. automethod:: curve4_to(location: Vec3, ctrl1: Vec3, ctrl2: Vec3)

    .. automethod:: close

    .. automethod:: clone() -> Path

    .. automethod:: reversed() -> Path

    .. automethod:: clockwise() -> Path

    .. automethod:: counter_clockwise() -> Path

    .. automethod:: transform(m: Matrix44) -> Path

    .. automethod:: approximate(segments: int=20) -> Iterable[Vec3]

    .. automethod:: flattening(distance: float, segments: int=16) -> Iterable[Vec3]

.. _PathPatch: https://matplotlib.org/3.1.1/api/_as_gen/matplotlib.patches.PathPatch.html#matplotlib.patches.PathPatch
.. _QPainterPath: https://doc.qt.io/qt-5/qpainterpath.html
.. _SVG-Path: https://developer.mozilla.org/en-US/docs/Web/SVG/Tutorial/Paths
.. _Context: https://pycairo.readthedocs.io/en/latest/reference/context.html