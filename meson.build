# If you are using Ubuntu/Xenial, Do "force-version" on meson to get the required version.
# If you are using Tizen 5.0+ or Ubuntu/Bionix+, you don't need to mind meson version.

project('nnstreamer', 'c', 'cpp',
  version: '1.5.2',
  license: ['LGPL'],
  meson_version: '>=0.50.0',
  default_options: [
    'werror=true',
    'warning_level=1',
    'c_std=gnu89',
    'cpp_std=c++14'
  ]
)

add_project_arguments('-DVERSION="' + meson.project_version() + '"', language: ['c', 'cpp'])

cc = meson.get_compiler('c')
cxx = meson.get_compiler('cpp')
build_platform = ''

if get_option('enable-tizen')
  # Pass __TIZEN__ to the compiler
  add_project_arguments('-D__TIZEN__=1', language: ['c', 'cpp'])
  build_platform = 'tizen'

  tizenVmajor = get_option('tizen-version-major')
  add_project_arguments('-DTIZENVERSION='+tizenVmajor.to_string(), language: ['c', 'cpp'])

elif not meson.is_cross_build()
  if cc.get_id() == 'clang' and cxx.get_id() == 'clang'
    if build_machine.system() == 'darwin'
      # Pass __MACOS__ to the compiler
      add_project_arguments('-D__MACOS__=1', language: ['c', 'cpp'])
      build_platform = 'macos'
    endif
  endif
endif

warning_flags = [
  '-Wredundant-decls',
  '-Wwrite-strings',
  '-Wformat',
  '-Wformat-nonliteral',
  '-Wformat-security',
  '-Winit-self',
  '-Waddress',
  '-Wno-multichar',
  '-Wvla',
  '-Wpointer-arith'
]

warning_c_flags = [
  '-Wmissing-declarations',
  '-Wmissing-include-dirs',
  '-Wmissing-prototypes',
  '-Wnested-externs',
  '-Waggregate-return',
  '-Wold-style-definition',
  '-Wdeclaration-after-statement'
]

foreach extra_arg : warning_flags
  if cc.has_argument (extra_arg)
    add_project_arguments([extra_arg], language: 'c')
  endif
  if cxx.has_argument (extra_arg)
    add_project_arguments([extra_arg], language: 'cpp')
  endif
endforeach

foreach extra_arg : warning_c_flags
  if cc.has_argument (extra_arg)
    add_project_arguments([extra_arg], language: 'c')
  endif
endforeach

gst_api_verision = '1.0'

# Set install path
nnstreamer_prefix = get_option('prefix')
nnstreamer_libdir = join_paths(nnstreamer_prefix, get_option('libdir'))
nnstreamer_bindir = join_paths(nnstreamer_prefix, get_option('bindir'))
nnstreamer_includedir = join_paths(nnstreamer_prefix, get_option('includedir'))
nnstreamer_inidir = get_option('sysconfdir')

# nnstreamer plugins path
plugins_install_dir = join_paths(nnstreamer_libdir, 'gstreamer-' + gst_api_verision)

# nnstreamer sub-plugins path
subplugin_install_prefix = join_paths(nnstreamer_prefix, 'lib', 'nnstreamer')
filter_subplugin_install_dir = join_paths(subplugin_install_prefix, 'filters')
decoder_subplugin_install_dir = join_paths(subplugin_install_prefix, 'decoders')
customfilter_install_dir = join_paths(subplugin_install_prefix, 'customfilters')
unittest_install_dir = join_paths(subplugin_install_prefix, 'unittest')

# Set default configuration
nnstreamer_conf = configuration_data()
nnstreamer_conf.set('VERSION', meson.project_version())
nnstreamer_conf.set('PREFIX', nnstreamer_prefix)
nnstreamer_conf.set('EXEC_PREFIX', nnstreamer_bindir)
nnstreamer_conf.set('LIB_INSTALL_DIR', nnstreamer_libdir)
nnstreamer_conf.set('GST_INSTALL_DIR', plugins_install_dir)
nnstreamer_conf.set('INCLUDE_INSTALL_DIR', nnstreamer_includedir)
nnstreamer_conf.set('SUBPLUGIN_INSTALL_PREFIX', subplugin_install_prefix)

# Set framework priority about model file extension when automatically selecting framework for tensor filter.
nnstreamer_conf.set('FRAMEWORK_PRIORITY_TFLITE', get_option('framework-priority-tflite'))

# Define default conf file
add_project_arguments('-DNNSTREAMER_CONF_FILE="' + join_paths(nnstreamer_inidir, 'nnstreamer.ini') + '"', language: 'c')

# Dependencies
glib_dep = dependency('glib-2.0')
gobject_dep = dependency('gobject-2.0')
gmodule_dep = dependency('gmodule-2.0')
gst_dep = dependency('gstreamer-' + gst_api_verision)
gst_base_dep = dependency('gstreamer-base-' + gst_api_verision)
gst_controller_dep = dependency('gstreamer-controller-' + gst_api_verision)
gst_video_dep = dependency('gstreamer-video-' + gst_api_verision)
gst_audio_dep = dependency('gstreamer-audio-' + gst_api_verision)
gst_app_dep = dependency('gstreamer-app-' + gst_api_verision)
gst_check_dep = dependency('gstreamer-check-' + gst_api_verision)

libm_dep = cc.find_library('m') # cmath library
libdl_dep = cc.find_library('dl') # DL library
thread_dep = dependency('threads') # pthread for tensorflow-lite

# Protobuf
protobuf_dep = dependency('protobuf', version: '>= 3.6.1', required: false)

# Orc
have_orcc = false

if get_option('enable-orc')
  orc_dep = dependency('orc-0.4', version: '>= 0.4.17', required: true)
  orcc = find_program('orcc', required: true)
  if orc_dep.found() and orcc.found()
    have_orcc = true
    orcc_args = [orcc, '--include', 'glib.h']
    add_project_arguments('-DHAVE_ORC=1', language: ['c', 'cpp'])
  else
    error('Cannot find orc library')
  endif
else
  add_project_arguments('-DDISABLE_ORC=1', language: ['c', 'cpp'])
endif

# NO Video support
if get_option('disable-video-support')
  add_project_arguments('-DNO_VIDEO=1', language: ['c', 'cpp'])
  message('Disable Video Type Support')
endif

# NO Audio support
if get_option('disable-audio-support')
  add_project_arguments('-DNO_AUDIO=1', language: ['c', 'cpp'])
  message('Disable Audio Type Support')
endif

# Tensorflow
if get_option('enable-tensorflow')
  tf_dep = dependency('tensorflow', required: true)

  if tf_dep.found() and protobuf_dep.found()
    add_project_arguments('-DENABLE_TENSORFLOW=1', language: ['c', 'cpp'])
  else
    error('Cannot find tensorflow')
  endif
endif

# Tensorflow-lite
if get_option('enable-tensorflow-lite')
  tflite_dep = dependency('tensorflow-lite', required: true)

  if tflite_dep.found()
    add_project_arguments('-DENABLE_TENSORFLOW_LITE=1', language: ['c', 'cpp'])
  else
    error('Cannot find tensorflow-lite')
  endif
endif

# PyTorch
if get_option('enable-pytorch')
  torch_dep = dependency('pytorch', required: true)

  if torch_dep.found()
    add_project_arguments('-DENABLE_PYTORCH=1', language: ['c', 'cpp'])
  else
    error('Cannot find pytorch')
  endif
endif

# Caffe2
if get_option('enable-caffe2')
  caffe2_dep = dependency('caffe2', required: true)

  if caffe2_dep.found() and protobuf_dep.found()
    add_project_arguments('-DENABLE_CAFFE2=1', language: ['c', 'cpp'])
  else
    error('Cannot find caffe2')
  endif
endif

# Python
have_python2 = false
have_python3 = false

if get_option('enable-python')
  pg_pkgconfig = find_program('pkg-config')
  # Check python 2.7
  python2_dep = dependency('python-2.7', required: false)
  if python2_dep.found()
    python2_inc_args = []
    python2_inc_args += run_command(pg_pkgconfig, ['python-2.7', '--cflags']).stdout().strip().split()
    python2_inc_args += run_command('python2.7', ['-c', 'import site\nfor i in site.getsitepackages(): print("-I" + i + "/numpy/core/include")']).stdout().strip().split()
    python2_inc_args += '-I' + run_command('python2.7', ['-m', 'site', '--user-site']).stdout().strip() + '/numpy/core/include'
    python2_inc_valid_args = []
    foreach python2_inc_arg : python2_inc_args
      if cxx.has_argument(python2_inc_arg) and \
          cxx.check_header('numpy/arrayobject.h', args: python2_inc_arg, dependencies : python2_dep)
        python2_inc_valid_args += python2_inc_arg
        have_python2 = true
        break
      endif
    endforeach
    if have_python2
      python2_deps = declare_dependency(dependencies: python2_dep,
          compile_args : python2_inc_valid_args)
    else
      warning('Found python2.7, but failed to find numpy.')
      warning('Disable nnstreamer-python2.')
    endif
  endif

  # Check python 3.x
  python3_dep = dependency('python3', required: false)
  if python3_dep.found()
    message('Finding numpy for python3')
    numpy_arryobj_h = 'numpy/arrayobject.h'

    python3_inc_args = []
    python3_inc_args += run_command(pg_pkgconfig, ['python3', '--cflags']).stdout().strip().split()
    python3_inc_args += run_command('python3', ['-c', 'import site\nfor i in site.getsitepackages(): print("-I" + i + "/numpy/core/include")']).stdout().strip().split()
    python3_inc_args += '-I' + run_command('python3', ['-m', 'site', '--user-site']).stdout().strip() + '/numpy/core/include'
    python3_inc_valid_args = []
    foreach python3_inc_arg : python3_inc_args
      if cxx.has_argument(python3_inc_arg) and \
          cxx.check_header('numpy/arrayobject.h', args : python3_inc_arg, dependencies : python3_dep)
        python3_inc_valid_args += python3_inc_arg
        have_python3 = true
        break
      endif
    endforeach
    if have_python3
      python3_deps = declare_dependency(dependencies: python3_dep,
          compile_args : python3_inc_valid_args)
    else
      warning('Found python3, but failed to find numpy.')
      warning('Disable nnstreamer-python3.')
    endif
  endif
endif

# nnfw-runtime ( details in https://review.tizen.org/gerrit/p/platform/core/ml/nnfw )
if get_option('enable-nnfw-runtime')
  add_project_arguments('-DENABLE_NNFW_RUNTIME=1', language: ['c', 'cpp'])
  # This enables nnfw as a subplugin for Tizen.
endif

# nnfw-runtime ( details in https://review.tizen.org/gerrit/p/platform/core/ml/nnfw )
if get_option('enable-tflite-nnapi-delegation')
  add_project_arguments('-DENABLE_TFLITE_NNAPI_DELEGATION=1', language: ['c', 'cpp'])
  # For tf-lite/nnapi, enable-nnfw allows to use nnfw::tflite::nnapi as a backend.
endif

# ArmNN
if get_option('enable-armnn')
  armnn_dep = dependency('armnn', required: true)
  add_project_arguments('-DENABLE_ARMNN=1', language: ['c', 'cpp'])
endif

# Movidius NCSDK2
have_movidius_ncsdk2 = false
if get_option('enable-movidius-ncsdk2')
  # Explicitly checks mvnc.h in the ncsdk2 directory
  nns_mvncsdk2_dep = cc.find_library('mvnc', required: false)
  if not nns_mvncsdk2_dep.found()
    warning('Failed to find \'libmvnc.so\' despite setting enable-movidius-ncsdk2. This option be ignored.')
  elif not cc.check_header('mvnc2/mvnc.h')
    warning('Failed to find \'mvnc2/mvnc.h\' despite setting enable-movidius-ncsdk2. This option be ignored.')
  else
    have_movidius_ncsdk2 = true
    add_project_arguments('-DENABLE_MOVIDIUS_NCSDK2=1', language: ['c', 'cpp'])
  endif
endif

# Set configuration to install .ini
nnstreamer_install_conf = configuration_data()
nnstreamer_install_conf.merge_from(nnstreamer_conf)

nnstreamer_install_conf.set('ENABLE_ENV_VAR', get_option('enable-env-var'))
nnstreamer_install_conf.set('ENABLE_SYMBOLIC_LINK', get_option('enable-symbolic-link'))
nnstreamer_install_conf.set('TORCH_USE_GPU', get_option('enable-pytorch-use-gpu'))

# Element restriction
restriction_config = ''

if get_option('enable-element-restriction')
  restriction_config = '''[element-restriction]
enable_element_restriction=True
restricted_elements=''' + get_option('restricted-elements')
endif

nnstreamer_install_conf.set('ELEMENT_RESTRICTION_CONFIG', restriction_config)

# Install .ini
configure_file(input: 'nnstreamer.ini.in', output: 'nnstreamer.ini',
  install_dir: nnstreamer_inidir,
  configuration: nnstreamer_install_conf
)

# Install .pc
configure_file(input: 'nnstreamer.pc.in', output: 'nnstreamer.pc',
  install_dir: join_paths(nnstreamer_libdir, 'pkgconfig'),
  configuration: nnstreamer_install_conf
)

# Build nnstreamer (common, plugins)
subdir('gst')

# Build ext subplugins
subdir('ext')

# Build API
subdir('api')

# Build nnstreamer examples
if get_option('enable-test') or get_option('install-example')
  subdir('nnstreamer_example')
endif

# Build unittests
if get_option('enable-test')
  subdir('tests')

  # temporary ini file for test, enable env variables.
  nnstreamer_test_conf = configuration_data()
  nnstreamer_test_conf.merge_from(nnstreamer_conf)

  nnstreamer_test_conf.set('ENABLE_ENV_VAR', true)
  nnstreamer_test_conf.set('ENABLE_SYMBOLIC_LINK', false)
  nnstreamer_test_conf.set('TORCH_USE_GPU', false)
  nnstreamer_test_conf.set('ELEMENT_RESTRICTION_CONFIG', '')

  configure_file(input: 'nnstreamer.ini.in', output: 'nnstreamer-test.ini',
    install: get_option('install-test'),
    install_dir: unittest_install_dir,
    configuration: nnstreamer_test_conf
  )
endif
