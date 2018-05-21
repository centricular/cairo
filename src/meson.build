cairo_sources = [
  'cairo-analysis-surface.c',
  'cairo-arc.c',
  'cairo-array.c',
  'cairo-atomic.c',
  'cairo-base64-stream.c',
  'cairo-base85-stream.c',
  'cairo-bentley-ottmann-rectangular.c',
  'cairo-bentley-ottmann-rectilinear.c',
  'cairo-bentley-ottmann.c',
  'cairo-botor-scan-converter.c',
  'cairo-boxes-intersect.c',
  'cairo-boxes.c',
  'cairo-cache.c',
  'cairo-clip-boxes.c',
  'cairo-clip-polygon.c',
  'cairo-clip-region.c',
  'cairo-clip-surface.c',
  'cairo-clip-tor-scan-converter.c',
  'cairo-clip.c',
  'cairo-color.c',
  'cairo-composite-rectangles.c',
  'cairo-compositor.c',
  'cairo-contour.c',
  'cairo-damage.c',
  'cairo-debug.c',
  'cairo-default-context.c',
  'cairo-device.c',
  'cairo-error.c',
  'cairo-fallback-compositor.c',
  'cairo-fixed.c',
  'cairo-font-face-twin-data.c',
  'cairo-font-face-twin.c',
  'cairo-font-face.c',
  'cairo-font-options.c',
  'cairo-freed-pool.c',
  'cairo-freelist.c',
  'cairo-gstate.c',
  'cairo-hash.c',
  'cairo-hull.c',
  'cairo-image-compositor.c',
  'cairo-image-info.c',
  'cairo-image-source.c',
  'cairo-image-surface.c',
  'cairo-line.c',
  'cairo-lzw.c',
  'cairo-mask-compositor.c',
  'cairo-matrix.c',
  'cairo-mempool.c',
  'cairo-mesh-pattern-rasterizer.c',
  'cairo-misc.c',
  'cairo-mono-scan-converter.c',
  'cairo-mutex.c',
  'cairo-no-compositor.c',
  'cairo-observer.c',
  'cairo-output-stream.c',
  'cairo-paginated-surface.c',
  'cairo-path-bounds.c',
  'cairo-path-fill.c',
  'cairo-path-fixed.c',
  'cairo-path-in-fill.c',
  'cairo-path-stroke-boxes.c',
  'cairo-path-stroke-polygon.c',
  'cairo-path-stroke-traps.c',
  'cairo-path-stroke-tristrip.c',
  'cairo-path-stroke.c',
  'cairo-path.c',
  'cairo-pattern.c',
  'cairo-pen.c',
  'cairo-polygon-intersect.c',
  'cairo-polygon-reduce.c',
  'cairo-polygon.c',
  'cairo-raster-source-pattern.c',
  'cairo-recording-surface.c',
  'cairo-rectangle.c',
  'cairo-rectangular-scan-converter.c',
  'cairo-region.c',
  'cairo-rtree.c',
  'cairo-scaled-font.c',
  'cairo-shape-mask-compositor.c',
  'cairo-slope.c',
  'cairo-spans-compositor.c',
  'cairo-spans.c',
  'cairo-spline.c',
  'cairo-stroke-dash.c',
  'cairo-stroke-style.c',
  'cairo-surface-clipper.c',
  'cairo-surface-fallback.c',
  'cairo-surface-observer.c',
  'cairo-surface-offset.c',
  'cairo-surface-snapshot.c',
  'cairo-surface-subsurface.c',
  'cairo-surface-wrapper.c',
  'cairo-surface.c',
  'cairo-time.c',
  'cairo-tor-scan-converter.c',
  'cairo-tor22-scan-converter.c',
  'cairo-toy-font-face.c',
  'cairo-traps-compositor.c',
  'cairo-traps.c',
  'cairo-tristrip.c',
  'cairo-unicode.c',
  'cairo-user-font.c',
  'cairo-version.c',
  'cairo-wideint.c',
  'cairo.c',
]

cairo_png_sources = [
  'cairo-png.c',
]

cairo_ft_sources = [
  'cairo-ft-font.c',
]

cairo_xlib_sources = [
  'cairo-xlib-display.c',
  'cairo-xlib-core-compositor.c',
  'cairo-xlib-fallback-compositor.c',
  'cairo-xlib-render-compositor.c',
  'cairo-xlib-screen.c',
  'cairo-xlib-source.c',
  'cairo-xlib-surface.c',
  'cairo-xlib-surface-shm.c',
  'cairo-xlib-visual.c',
  'cairo-xlib-xcb-surface.c',
]

cairo_no_warn_c_args = cc.get_supported_arguments([
  '-Wno-attributes',
  '-Wno-unused-but-set-variable',
  '-Wno-missing-field-initializers',
  '-Wno-unused-parameter',
  '-Wno-long-long',
])

incsrc = include_directories('.')

if feature_conf.get('CAIRO_HAS_PNG_FUNCTIONS', 0) == 1
  cairo_sources += cairo_png_sources
endif

if feature_conf.get('CAIRO_HAS_FT_FONT', 0) == 1
  cairo_sources += cairo_ft_sources
endif

if feature_conf.get('CAIRO_HAS_XLIB_SURFACE', 0) == 1
  cairo_sources += cairo_xlib_sources
endif

libcairo = library('cairo', cairo_sources,
  dependencies: deps,
  c_args: cairo_no_warn_c_args + pthread_c_args + ['-DHAVE_CONFIG_H'],
  link_args: pthread_link_args,
  include_directories: incbase,
)

configure_file(output: 'cairo-features.h', configuration: feature_conf)
