project('cairo', 'c', 'cpp')

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

pixman_dep = dependency('pixman-1')
m_dep = cc.find_library('libm')
png_dep = cc.find_library('libpng')
fontconfig_dep = dependency('fontconfig')
freetype_dep = dependency('freetype2')
zlib_dep = dependency('zlib', required: false)
lzo_dep = dependency('lzo2', required: false)
dl_dep = cc.find_library('dl', required: false)
x11_dep = dependency('x11', required: false)
xext_dep = dependency('xext', required: false)

deps = [pixman_dep, m_dep, png_dep]

conf = configuration_data()

if zlib_dep.found()
  deps += [zlib_dep]
  conf.set('HAVE_ZLIB', 1)
endif

if lzo_dep.found()
  deps += [lzo_dep]
  conf.set('HAVE_LZO', 1)
endif

if dl_dep.found() and cc.has_function('dlsym', dependencies: [dl_dep])
  deps += [dl_dep]
  conf.set('CAIRO_HAS_DLSYM', 1)
elif cc.has_function('dlsym')
  conf.set('CAIRO_HAS_DLSYM', 1)
elif cc.has_function('dlsym', prefix: '#include <dlfcn.h>')
  conf.set('CAIRO_HAS_DLSYM', 1)
endif

feature_conf = configuration_data()

if freetype_dep.found()
  feature_conf.set('CAIRO_HAS_FT_FONT', 1)
  check_funcs += [
    'FT_Get_X11_Font_Format',
    'FT_GlyphSlot_Embolden',
    'FT_GlyphSlot_Oblique',
    'FT_Load_Sfnt_Table',
    'FT_Library_SetLcdFilter',
    'FT_Get_Var_Design_Coordinates',
    'FT_Done_MM_Var',
  ]

  conf.set('FT_HAS_COLOR', cc.links(files('meson-cc-tests/ft_has_color.c'), dependencies: freetype_dep, name: 'FT has color'))

  deps += [freetype_dep]
endif

if fontconfig_dep.found()
  feature_conf.set('CAIRO_HAS_FC_FONT', 1)
  check_funcs += [
    'FcInit',
    'FcFini'
  ]

  deps += [fontconfig_dep]
endif

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

  xrender_dep = dependency('xrender')

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

feature_conf.set('CAIRO_FEATURES_H', true)
feature_conf.set10('CAIRO_HAS_PNG_FUNCTIONS', png_dep.found())
feature_conf.set10('CAIRO_HAS_FC_FONT', fontconfig_dep.found())
feature_conf.set('CAIRO_HAS_USER_FONT', 1)
feature_conf.set('CAIRO_HAS_IMAGE_SURFACE', 1)

feature_conf.set('CAIRO_HAS_MIME_SURFACE', 1)
feature_conf.set('CAIRO_HAS_RECORDING_SURFACE', 1)
feature_conf.set('CAIRO_HAS_OBSERVER_SURFACE', 1)

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

if cc.links(files('meson-cc-tests/atomic-ops-cxx11.c'), name: 'Atomic ops: cxx11')
  conf.set('HAVE_CXX11_ATOMIC_PRIMITIVES', 1)
elif cc.links(files('meson-cc-tests/atomic-ops-gcc-legacy.c'), name: 'Atomic ops: gcc legacy')
  conf.set('HAVE_GCC_LEGACY_ATOMICS', 1)
elif cc.has_header('atomic_ops.h')
  conf.set('HAVE_LIB_ATOMIC_OPS', 1)
elif cc.has_header('libkern/OSAtomic.h')
  conf.set('HAVE_OS_ATOMIC_OPS', 1)
endif

if not ['x86', 'x86_64'].contains(host_machine.cpu_family())
  conf.set('ATOMIC_OP_NEEDS_MEMORY_BARRIER', 1)
endif

subdir('src')

if feature_conf.get('CAIRO_HAS_PNG_FUNCTIONS', 0) == 1
  subdir('boilerplate')
  subdir('test')
endif

configure_file(output: 'config.h', configuration: conf)
