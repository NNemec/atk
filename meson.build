project('atk', 'c',
        version: '2.31.92',
        license: 'LGPLv2.1+',
        default_options: [
          'buildtype=debugoptimized',
          'warning_level=1',
          'c_std=c99',
        ],
        meson_version : '>= 0.46.0')

cc = meson.get_compiler('c')
host_system = host_machine.system()

version = meson.project_version().split('.')
atk_major_version = version[0].to_int()
atk_minor_version = version[1].to_int()
atk_micro_version = version[2].to_int()

atk_interface_age = 1
atk_binary_age = 10000 * atk_major_version + 100 * atk_minor_version + 10 + atk_micro_version

atk_api_version = '1.0'
atk_api_path = 'atk-@0@/atk'.format(atk_api_version)

atk_prefix = get_option('prefix')
atk_libdir = join_paths(atk_prefix, get_option('libdir'))
atk_sysconfdir = join_paths(atk_prefix, get_option('sysconfdir'))
atk_includedir = join_paths(atk_prefix, get_option('includedir'))
atk_datadir = join_paths(atk_prefix, get_option('datadir'))
atk_libexecdir = join_paths(atk_prefix, get_option('libexecdir'))

atk_conf = configuration_data()

atk_conf.set_quoted('VERSION', meson.project_version())
atk_conf.set_quoted('GETTEXT_PACKAGE', 'atk10')

# Maintain version scheme with libtool
atk_soversion = 0
current = atk_binary_age - atk_interface_age
atk_libversion = '@0@.@1@.@2@'.format(atk_soversion, current, atk_interface_age)
darwin_versions = ['@0@'.format(current + 1), '@0@.@1@'.format(current + 1, atk_interface_age)]

add_project_arguments([ '-DG_DISABLE_SINGLE_INCLUDES', '-DATK_DISABLE_SINGLE_INCLUDES' ], language: 'c')

if cc.get_id() == 'msvc'
  add_project_arguments([ '-FImsvc_recommended_pragmas.h' ], language: 'c')
endif

# Compiler and linker flags
common_cflags = []
common_ldflags = []

test_cflags = []

# Symbol visibility
if get_option('default_library') != 'static'
  if host_system == 'windows'
    atk_conf.set('DLL_EXPORT', true)
    atk_conf.set('_ATK_EXTERN', '__declspec(dllexport) extern')
    if cc.get_id() != 'msvc'
      test_cflags += ['-fvisibility=hidden']
    endif
  else
    atk_conf.set('_ATK_EXTERN', '__attribute__((visibility("default"))) extern')
    test_cflags += ['-fvisibility=hidden']
  endif
endif

# Check all compiler flags
common_cflags += cc.get_supported_arguments(test_cflags)

# Linker flags
if host_machine.system() == 'linux'
  test_ldflags = [ '-Wl,-Bsymbolic', '-Wl,-z,relro', '-Wl,-z,now', ]
  common_ldflags += cc.get_supported_link_arguments(test_ldflags)
endif

# Maintain compatibility with autotools on macOS
if host_machine.system() == 'darwin'
  common_ldflags += [ '-compatibility_version', darwin_versions[0], '-current_version', darwin_versions[1]]
endif

# Functions
checked_funcs = [
  'bind_textdomain_codeset',
]

foreach f: checked_funcs
  if cc.has_function(f)
    atk_conf.set('HAVE_' + f.underscorify().to_upper(), 1)
  endif
endforeach

# Dependencies
glib_req_version = '>= 2.31.2'

glib_dep = [dependency('glib-2.0', version: glib_req_version,
                          fallback : ['glib', 'libglib_dep']),
            dependency('gobject-2.0', version: glib_req_version,
                          fallback : ['glib', 'libgobject_dep'])]

# Compat variables for pkgconfig
pkgconf = configuration_data()
pkgconf.set('prefix', atk_prefix)
pkgconf.set('exec_prefix', atk_prefix)
pkgconf.set('libdir', atk_libdir)
pkgconf.set('includedir', atk_includedir)
pkgconf.set('VERSION', meson.project_version())
pkgconf.set('ATK_API_VERSION', atk_api_version)
pkgconf.set('srcdir', '.')

foreach pkg: [ 'atk.pc', ]
  configure_file(input: pkg + '.in',
                 output: pkg,
                 configuration: pkgconf,
                 install: true,
                 install_dir: join_paths(atk_libdir, 'pkgconfig'))
endforeach

gnome = import('gnome')

# Internal configuration header
configure_file(output: 'config.h', configuration: atk_conf)

root_inc = include_directories('.')

subdir('atk')
subdir('tests')

if find_program('xgettext', required : false).found()
  subdir('po')
endif

if get_option('docs')
  subdir('docs')
endif
