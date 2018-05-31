project('cairo', 'c', 'cpp',
  meson_version: '> 0.46.99',
)

cc = meson.get_compiler('c')
python3 = import('python').find_installation()

check_sizeofs = [
  ['void *', {'conf-name': 'SIZEOF_VOID_P'}],
  ['int'],
  ['long'],
  ['long long'],
  ['size_t'],
]

check_headers = [
  ['stdint.h'],
  ['inttypes.h'],
  ['sys/int_types.h'],
  ['fcntl.h'],
  ['unistd.h'],
  ['signal.h'],
  ['sys/stat.h'],
  ['sys/socket.h'],
  ['sys/poll.h'],
  ['sys/un.h'],
  ['sched.h', {'check-funcs': ['sched_getaffinity']}],
  ['sys/mman.h', {'check-funcs': ['mmap']}],
  ['time.h', {'check-funcs': ['clock_gettime']}],
  ['libgen.h'],
  ['byteswap.h'],
  ['signal.h'],
  ['setjmp.h'],
  ['fenv.h'],
  ['sys/wait.h'],
  ['sys/stat.h'],
  ['io.h'],
  ['fenv.h', {'check-funcs': ['feenableexcept', 'fedisableexcept', 'feclearexcept']}],
  ['xlocale.h'],
  ['sys/ioctl.h'],
]

check_types = [
  ['uint64_t', {'headers': ['stdint.h']}],
  ['uint128_t', {'headers': ['stdint.h']}],
  ['__uint128_t']
]

check_funcs = [
  'alarm',
  'ctime_r',
  'localtime_r',
  'gmtime_r',
  'drand48',
  'flockfile',
  'funlockfile',
  'getline',
  'link',
  'strndup',
  'fork',
  'waitpid',
  'raise',
  'newlocale',
  'strtod_l',
]

check_thread_flags = [
  [['-D_REENTRANT'], ['-lpthread']],
  [['-pthread'], []],
  [['-D_REENTRANT'], [], {'real': false}],
]

m_dep = cc.find_library('libm', required: false)
# Used in util
gtk_dep = dependency('gtk+-2.0', required: false)

deps = [m_dep]
internal_deps = []
extra_link_args = []

conf = configuration_data()

lzo_dep = dependency('lzo2', required: false)
if lzo_dep.found()
  deps += [lzo_dep]
  conf.set('HAVE_LZO', 1)
endif

dl_dep = cc.find_library('dl', required: false)
if dl_dep.found() and cc.has_function('dlsym', dependencies: [dl_dep])
  deps += [dl_dep]
  conf.set('CAIRO_HAS_DLSYM', 1)
elif cc.has_function('dlsym')
  conf.set('CAIRO_HAS_DLSYM', 1)
elif cc.has_function('dlsym', prefix: '#include <dlfcn.h>')
  conf.set('CAIRO_HAS_DLSYM', 1)
endif

feature_conf = configuration_data()

zlib_dep = dependency('zlib',
  required: false,
  fallback : ['zlib', 'zlib_dep'],
)
if zlib_dep.found()
  if zlib_dep.type_name() == 'internal'
    internal_deps += [zlib_dep]
  else
    deps += [zlib_dep]
  endif
  conf.set('HAVE_ZLIB', 1)
endif

png_dep = dependency('libpng',
  required: false,
  fallback: ['libpng', 'png_dep']
)
if png_dep.found()
  feature_conf.set('CAIRO_HAS_SVG_SURFACE', 1)
  feature_conf.set('CAIRO_HAS_PNG_FUNCTIONS', 1)
  if png_dep.type_name() == 'internal'
    internal_deps += [png_dep]
  else
    deps += [png_dep]
  endif
endif

freetype_dep = dependency('freetype2',
  required: false,
  version: '>= 9.7.3',
  fallback: ['freetype2', 'freetype_dep'],
)
if freetype_dep.found()
  feature_conf.set('CAIRO_HAS_FT_FONT', 1)
  ft_check_funcs = [
    'FT_Get_X11_Font_Format',
    'FT_GlyphSlot_Embolden',
    'FT_GlyphSlot_Oblique',
    'FT_Load_Sfnt_Table',
    'FT_Library_SetLcdFilter',
    'FT_Get_Var_Design_Coordinates',
    'FT_Done_MM_Var',
  ]

  if freetype_dep.type_name() == 'internal'
    foreach func : ft_check_funcs
      conf.set('HAVE_@0@'.format(func.to_upper()), 1)
    endforeach
    internal_deps += [freetype_dep]
  else
    if not cc.links(files('meson-cc-tests/ft_has_color.c'), dependencies: freetype_dep, name: 'FT has color')
      conf.set('FT_HAS_COLOR', '(0)')
    endif
    check_funcs += ft_check_funcs
    deps += [freetype_dep]
  endif
endif

fontconfig_dep = dependency('fontconfig',
  required: false,
  version: '>= 2.2.95',
  fallback: ['fontconfig', 'fontconfig_dep'],
)
if fontconfig_dep.found()
  feature_conf.set('CAIRO_HAS_FC_FONT', 1)
  feature_conf.set('CAIRO_HAS_FC_FONT', 1)
  fc_check_funcs = [
    'FcInit',
    'FcFini'
  ]

  if fontconfig_dep.type_name() == 'internal'
    foreach func : fc_check_funcs
      conf.set('HAVE_@0@'.format(func.to_upper()), 1)
    endforeach
    internal_deps += [fontconfig_dep]
  else
    check_funcs += fc_check_funcs
    deps += [fontconfig_dep]
  endif
endif

x11_dep = dependency('x11', required: false)
xext_dep = dependency('xext', required: false)
if x11_dep.found() and xext_dep.found()
  feature_conf.set('CAIRO_HAS_XLIB_SURFACE', 1)
  extra_headers = ['X11/Xlibint.h', 'X11/Xproto.h']
  check_headers += [
    ['X11/extensions/XShm.h', {'extra-headers': extra_headers}],
    ['X11/extensions/shmproto.h', {'extra-headers': extra_headers}],
    ['X11/extensions/shmstr.h', {'extra-headers': extra_headers}],
  ]
  deps += [x11_dep, xext_dep]
  res = cc.run(files('meson-cc-tests/ipc_rmid_deferred_release.c'),
    dependencies: [x11_dep, xext_dep],
    name: 'shmctl IPC_RMID allowes subsequent attaches')

  if res.returncode() == 0
    conf.set('IPC_RMID_DEFERRED_RELEASE', 1)
  endif
endif

if feature_conf.get('CAIRO_HAS_XLIB_SURFACE', 0) == 1
  xrender_dep = dependency('xrender', required: false, version: '>= 0.6')

  if xrender_dep.found()
    feature_conf.set('CAIRO_HAS_XLIB_XRENDER_SURFACE', 1)

    check_funcs += [
      'XRenderCreateSolidFill',
      'XRenderCreateLinearGradient',
      'XRenderCreateRadialGradient',
      'XRenderCreateConicalGradient',
    ]

    deps += [xrender_dep]
  endif
endif

xcb_dep = dependency('xcb', required: false, version: '>= 1.6')
xcb_render_dep = dependency('xcb-render', required: false, version: '>= 1.6')
if xcb_dep.found() and xcb_render_dep.found()
  feature_conf.set('CAIRO_HAS_XCB_SURFACE', 1)
  deps += [xcb_dep, xcb_render_dep]
endif

if feature_conf.get('CAIRO_HAS_XCB_SURFACE', 0) == 1 and feature_conf.get('CAIRO_HAS_XLIB_SURFACE', 0) == 1
  x11xcb_dep = dependency('x11-xcb', required: false)
  if x11xcb_dep.found()
    feature_conf.set('CAIRO_HAS_XLIB_XCB_FUNCTIONS', 1)
    deps += [x11xcb_dep]
  endif
endif

if feature_conf.get('CAIRO_HAS_XCB_SURFACE', 0) == 1
  xcbshm_dep = dependency('xcb-shm', required: false)
  if xcbshm_dep.found()
    feature_conf.set('CAIRO_HAS_XCB_SHM_FUNCTIONS', 1)
    deps += [xcbshm_dep]
  endif
endif

# Tests crash and burn
qtgui_dep = dependency('QtGui', required: false, version: '>= 4.4.0')
if qtgui_dep.found() and false
  feature_conf.set('CAIRO_HAS_QT_SURFACE', 1)
  deps += [qtgui_dep]
  extra_link_args += ['-lstdc++']
endif

# Untested
if cc.has_header('ApplicationServices/ApplicationServices.h')
  if cc.has_header('CoreGraphics/CoreGraphics.h')
    extra_link_args += ['-Xlinker', '-framework', '-Xlinker', 'CoreGraphics']
  else
    extra_link_args += ['-Xlinker', '-framework', '-Xlinker', 'ApplicationServices']
  endif

  feature_conf.set('CAIRO_HAS_QUARTZ_SURFACE', 1)
  feature_conf.set('CAIRO_HAS_QUARTZ_FONT', 1)
  feature_conf.set('CAIRO_HAS_QUARTZ_IMAGE_SURFACE', 1)
endif

if host_machine.system() == 'windows'
  feature_conf.set('CAIRO_HAS_WIN32_SURFACE', 1)
  deps += [
    cc.find_library('gdi32'),
    cc.find_library('msimg32'),
  ]
endif

if feature_conf.get('CAIRO_HAS_WIN32_SURFACE', 0) == 1
  feature_conf.set('CAIRO_HAS_WIN32_FONT', 1)
endif

# Doesn't build since at least 2011
libudev_dep = dependency('libudev', required: false, version: '>= 136')
libdrm_dep = dependency('libdrm', required: false, version: '>= 2.4')
if libudev_dep.found() and libdrm_dep.found() and false
  feature_conf.set('CAIRO_HAS_DRM_SURFACE', 1)
  deps += [libudev_dep, libdrm_dep]
endif

# Not even trying to port gallium as it depends on drm

# Not porting BEOS or OS/2 backends either, meson doesn't support those

# GL / GLESV2 / GLESV3 are mutually exclusive
use_gl = false
need_egl_functions = false
need_wgl_functions = false
need_glx_functions = false
gl_dep = dependency('gl', required: false)
if not gl_dep.found()
  if cc.has_header('GL/gl.h')
    extra_link_args += ['-lGL']
    use_gl = true
  endif
elif cc.has_header('GL/gl.h', dependencies: gl_dep) and cc.has_header('GL/glext.h', dependencies: gl_dep)
  use_gl = true
  deps += [gl_dep]
endif

if use_gl
  feature_conf.set('CAIRO_HAS_GL_SURFACE', 1)
  need_egl_functions = true
  need_wgl_functions = true
  need_glx_functions = true
endif

if feature_conf.get('CAIRO_HAS_GL_SURFACE', 0) == 0
  use_glesv2 = false
  glesv2_dep = dependency('glesv2', required: false)
  if not glesv2_dep.found()
    if cc.has_header('GLES2/gl2.h')
      use_glesv2 = true
      extra_link_args += ['-lGLESv2']
    endif
  else
    use_glesv2 = true
  endif

  if use_glesv2
    feature_conf.set('CAIRO_HAS_GLESV2_SURFACE', 1)
    need_egl_functions = true
  endif
endif

if feature_conf.get('CAIRO_HAS_GL_SURFACE', 0) == 0 and feature_conf.get('CAIRO_HAS_GLESV2_SURFACE', 0) == 0
  use_glesv3 = false
  glesv3_dep = dependency('glesv3', required: false)
  if not glesv3_dep.found()
    if cc.has_header('GLES3/gl3.h')
      use_glesv3 = true
      extra_link_args += ['-lGLESv2']
    endif
  else
    use_glesv3 = true
  endif

  if use_glesv3
    feature_conf.set('CAIRO_HAS_GLESV3_SURFACE', 1)
    need_egl_functions = true
  endif
endif

# Broken for me, depends on experimental API
cogl_dep = dependency('cogl-2.0-experimental', required: false)
if cogl_dep.found() and false
  feature_conf.set('CAIRO_HAS_COGL_SURFACE', 1)
  deps += [cogl_dep]
endif

# Untested
directfb_dep = dependency('directfb', required: false)
if directfb_dep.found()
  feature_conf.set('CAIRO_HAS_DIRECTFB_SURFACE', 1)
  deps += [directfb_dep]
endif

# Untested
if cc.has_header('VG/openvg.h')
  extra_link_args += ['-lOpenVG']
  feature_conf.set('CAIRO_HAS_VG_SURFACE', 1)
  need_egl_functions = true
  need_glx_functions = true
endif

if need_egl_functions
  egl_dep = dependency('egl', required: false)
  use_egl = false

  if not egl_dep.found()
    if cc.has_header('EGL/egl.h')
      egl_extra_deps = []
      csi_dep = cc.find_library('csi', required: false)
      if csi_dep.found() and cc.has_function('csi_stream_attachresource', dependencies: [csi_dep])
	egl_extra_deps = [csi_dep]
      endif

      foreach libname : ['EGL', 'egl13', 'egl12', 'egl11']
	if not use_egl
	  dep = cc.find_library(libname, required: false)
	  if dep.found() and cc.has_function('eglGetError', dependencies: [dep])
	    deps += [dep] + egl_extra_deps
	    use_egl = true
	  endif
	endif
      endforeach
    endif
  else
    use_egl = true
    deps += [egl_dep]
  endif

  if use_egl
    feature_conf.set('CAIRO_HAS_EGL_FUNCTIONS', 1)
  endif
endif

if need_glx_functions
  if cc.has_header('GL/glx.h')
    feature_conf.set('CAIRO_HAS_GLX_FUNCTIONS', 1)
  endif
endif

# Untested
if need_wgl_functions
  if cc.has_header('windows.h')
    feature_conf.set('CAIRO_HAS_WGL_FUNCTIONS', 1)
  endif
endif

gobject_dep = dependency('gobject-2.0',
  required: false,
  fallback: ['glib', 'libgobject_dep']
)
glib_dep = dependency('glib-2.0',
  required: false,
  version: '>= 2.14',
  fallback: ['glib', 'libglib_dep'],
)
if gobject_dep.found() and glib_dep.found()
  feature_conf.set('CAIRO_HAS_GOBJECT_FUNCTIONS', 1)
endif

if zlib_dep.found()
  feature_conf.set('CAIRO_HAS_SCRIPT_SURFACE', 1)
endif

if zlib_dep.found()
  feature_conf.set('CAIRO_HAS_PS_SURFACE', 1)
endif

if zlib_dep.found()
  feature_conf.set('CAIRO_HAS_PDF_SURFACE', 1)
endif

if zlib_dep.found()
  feature_conf.set('CAIRO_HAS_INTERPRETER', 1)
endif

if zlib_dep.found()
  feature_conf.set('CAIRO_HAS_XML_SURFACE', 1)
endif

# Untested, libiberty.h is in a libiberty subfolder for me
bfd_dep = cc.find_library('bfd', required: false)
if bfd_dep.found() and cc.has_function('bfd_openr', dependencies: [bfd_dep])
  if cc.has_header('libiberty.h')
    conf.set('HAVE_BFD', 1)
    deps += [bfd_dep]
  endif
endif

# Untested, see above
if conf.get('HAVE_BFD', 0) == 1
  conf.set('CAIRO_HAS_SYMBOL_LOOKUP', 1)
endif

if feature_conf.get('CAIRO_HAS_PS_SURFACE', 0) == 1
  gs = find_program('gs', required: false)
  libspectre_dep = dependency('libspectre', required: false, version: '>= 0.2.0')
  if gs.found() and libspectre_dep.found()
    conf.set('CAIRO_CAN_TEST_PS_SURFACE', 1)
  endif

  if libspectre_dep.found()
    conf.set('CAIRO_HAS_SPECTRE', 1)
    deps += [libspectre_dep]
  endif
endif

if feature_conf.get('CAIRO_HAS_PDF_SURFACE', 0) == 1
  poppler_dep = dependency('poppler-glib', required: false, version: '>= 0.17.4')
  if poppler_dep.found() and cc.has_function('poppler_page_render', dependencies: [poppler_dep])
    conf.set('CAIRO_CAN_TEST_PDF_SURFACE', 1)
    deps += [poppler_dep]
  endif
endif

if feature_conf.get('CAIRO_HAS_SVG_SURFACE', 0) == 1
  librsvg_dep = dependency('librsvg-2.0', required: false, version: '>= 2.35.0')
  if librsvg_dep.found() and cc.has_function('rsvg_pixbuf_from_file', dependencies: librsvg_dep)
    conf.set('CAIRO_CAN_TEST_SVG_SURFACE', 1)
    deps += [librsvg_dep]
  endif
endif

pixman_dep = dependency('pixman-1',
  required: false,
  version: '>= 0.30.0',
  fallback: ['pixman', 'pixman_dep'],
)
if pixman_dep.found()
  feature_conf.set('CAIRO_HAS_IMAGE_SURFACE', 1)
  conf.set('HAS_PIXMAN_GLYPHS', 1)
  if pixman_dep.type_name() == 'internal'
    internal_deps += [pixman_dep]
  else
    deps += [pixman_dep]
  endif
endif

feature_conf.set('CAIRO_FEATURES_H', true)
feature_conf.set('CAIRO_HAS_USER_FONT', 1)

feature_conf.set('CAIRO_HAS_MIME_SURFACE', 1)
feature_conf.set('CAIRO_HAS_RECORDING_SURFACE', 1)
feature_conf.set('CAIRO_HAS_OBSERVER_SURFACE', 1)
feature_conf.set('CAIRO_HAS_TEE_SURFACE', 1)

incbase = include_directories('.')

foreach check : check_sizeofs
  type = check[0]
  opts = check.length() > 1 ? check[1] : {}

  conf_name = opts.get('conf-name', 'SIZEOF_@0@'.format(type.underscorify().to_upper()))

  conf.set(conf_name, cc.sizeof(type))
endforeach

foreach check : check_headers
  name = check[0]
  opts = check.length() > 1 ? check[1] : {}
  prefix = ''

  foreach header : opts.get('extra-headers', [])
    prefix += '#include <@0@>\n'.format(header)
  endforeach

  if cc.has_header(name, prefix: prefix)
    conf.set('HAVE_@0@'.format(name.to_upper().underscorify()), 1)
    check_funcs += check.length() > 1 ? check[1].get('check-funcs', []) : []
  endif
endforeach

foreach check : check_types
  name = check[0]
  opts = check.length() > 1 ? check[1] : {}
  prefix = ''

  foreach header : opts.get('headers', [])
    prefix += '#include <@0@>\n'.format(header)
  endforeach

  if cc.has_type(name, prefix: prefix)
    conf.set('HAVE_@0@'.format(name.to_upper()), 1)
  endif
endforeach

foreach name : check_funcs
  if cc.has_function(name, dependencies: deps)
    conf.set('HAVE_@0@'.format(name.to_upper()), 1)
  endif
endforeach

pthread_c_args = []
pthread_link_args = []

foreach thread_flags : check_thread_flags
  if not conf.has('CAIRO_HAS_PTHREAD')
    cflags = thread_flags[0]
    lflags = thread_flags[1]
    real_pthread = thread_flags.length() > 2 ? thread_flags[2].get('real', true) : true

    if cc.links(files('meson-cc-tests/pthread.c'), args: cflags + lflags, name: 'pthreads')
      conf.set('CAIRO_HAS_PTHREAD', 1)
      if real_pthread
        conf.set('CAIRO_HAS_REAL_PTHREAD', 1)
      endif
      pthread_c_args = cflags
      pthread_link_args = lflags
    endif
  endif
endforeach

extra_link_args += pthread_link_args

if cc.links(files('meson-cc-tests/atomic-ops-cxx11.c'), name: 'Atomic ops: cxx11')
  conf.set('HAVE_CXX11_ATOMIC_PRIMITIVES', 1)
elif cc.links(files('meson-cc-tests/atomic-ops-gcc-legacy.c'), name: 'Atomic ops: gcc legacy')
  conf.set('HAVE_GCC_LEGACY_ATOMICS', 1)
elif cc.has_header('atomic_ops.h')
  conf.set('HAVE_LIB_ATOMIC_OPS', 1)
elif cc.has_header('libkern/OSAtomic.h')
  conf.set('HAVE_OS_ATOMIC_OPS', 1)
endif

test_mkdir_c_args = []
if conf.get('HAVE_SYS_STAT_H', 0) == 1
  test_mkdir_c_args += ['-DHAVE_SYS_STAT_H']
endif

if conf.get('HAVE_IO_H', 0) == 1
  test_mkdir_c_args += ['-DHAVE_IO_H']
endif

if cc.links(files('meson-cc-tests/mkdir-variant-1.c'), args: test_mkdir_c_args)
  conf.set('HAVE_MKDIR', 1)
elif cc.links(files('meson-cc-tests/mkdir-variant-2.c'), args: test_mkdir_c_args)
  conf.set('HAVE_MKDIR', 2)
else
  conf.set('HAVE_MKDIR', 0)
endif

if not ['x86', 'x86_64'].contains(host_machine.cpu_family())
  conf.set('ATOMIC_OP_NEEDS_MEMORY_BARRIER', 1)
endif

have_ld_preload = ['linux', 'freebsd', 'darwin', 'dragonfly'].contains(host_machine.system())

if have_ld_preload and zlib_dep.found() and conf.get('CAIRO_HAS_REAL_PTHREAD', 0) == 1 and conf.get('CAIRO_HAS_DLSYM', 0) == 1
  conf.set('CAIRO_HAS_TRACE', 1)
endif

rt_dep = cc.find_library('rt', required: false)
have_shm = false
if rt_dep.found() and cc.has_function('shm_open', dependencies: [rt_dep])
  have_shm = true
endif

# This to make sure we don't run checks against internal deps
deps += internal_deps

subdir('src')

if feature_conf.get('CAIRO_HAS_PNG_FUNCTIONS', 0) == 1
  subdir('boilerplate')
endif

subdir('util')

if feature_conf.get('CAIRO_HAS_PNG_FUNCTIONS', 0) == 1
  subdir('test')
endif

configure_file(output: 'config.h', configuration: conf)
