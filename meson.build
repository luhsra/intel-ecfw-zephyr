project('intel-ecfw-zephyr', 'cpp',
    default_options : ['warning_level=3',
                       'cpp_std=c++17',
                       'buildtype=debugoptimized'])

llvm_version = ['>=14', '<16']
llvm_dep = dependency('llvm', method: 'config-tool', version: llvm_version, include_type: 'system')

py3_mod = import('python')
py3_inst = py3_mod.find_installation('python3', modules: ['pykwalify', 'elftools'])

fs = import('fs')
zephyr = subproject('zephyr-toolchain')
zephyr_source_dir = zephyr.get_variable('zephyr_source_dir')
zephyr_work_dir = zephyr.get_variable('zephyr_work_dir')
zephyr_modules = ';'.join(zephyr.get_variable('modules'))

toolchains = subproject('toolchains')
ara_zephyr_apps = subproject('ara-zephyr-apps')

llvm_ld = toolchains.get_variable('llvm_ld')
llvm_objcopy = toolchains.get_variable('llvm_objcopy')
llvm_objdump = toolchains.get_variable('llvm_objdump')
llvm_nm = toolchains.get_variable('llvm_nm')
llvm_ar = toolchains.get_variable('llvm_ar')
llc = toolchains.get_variable('llc')
clang = toolchains.get_variable('clang')
clangpp = toolchains.get_variable('clangpp')
ara_link = toolchains.get_variable('ara_link')

cmake_tool = find_program('cmake')
ninja = find_program('ninja')

# not directly used within Meson but needed by Zephyr
dtc = find_program('dtc')

compile_zephyr_app = ara_zephyr_apps.get_variable('compile_zephyr_app')

zephyr_targets = []

if toolchains.get_variable('arm_enabled')
    cflags = '-mfloat-abi=soft -Wno-ignored-attributes'
    ldflags = '-fuse-ld=@0@ -L @1@'.format(toolchains.get_variable('arm_ld').full_path(),toolchains.get_variable('arm_libgcc_dir'))

    tgt_name = 'intel-ecfw-zephyr'
    board_name = 'mec172xmodular_assy6930'

    app = meson.current_source_dir()

    target = custom_target(tgt_name,
        output: [tgt_name + '.ll', tgt_name + '.config'],
        depends: ara_link,
        command: [
        py3_inst, compile_zephyr_app,
        '--cmake-builddir', meson.current_build_dir() / tgt_name + '_build',
        '--app-dir', app,
        '--cmake-program', cmake_tool.full_path(),
        '--ninja-program', ninja.full_path(),
        '--zephyr-dir', zephyr_work_dir / 'zephyr',
        '--target', 'app/libapp.ll',
        '--output', '@OUTPUT0@',
        '--kconfig', '@OUTPUT1@',
        '--cmake-args', ['BOARD=' + board_name,
                            'ZEPHYR_TOOLCHAIN_VARIANT=llvm',
                            'TOOLCHAIN_ROOT=' + zephyr_work_dir / 'zephyr',
                            'ZEPHYR_BASE=' + zephyr_work_dir / 'zephyr',
                            'ZEPHYR_MODULES=' + zephyr_modules,
                            'CMAKE_C_COMPILER=' + clang.full_path(),
                            'CMAKE_CXX_COMPILER=' + clangpp.full_path(),
                            'CMAKE_LLC=' + llc.full_path(),
                            'CMAKE_LLVM_LINK=' + ara_link.full_path(),
                            'CMAKE_OBJCOPY=' + llvm_objcopy.full_path(),
                            'CMAKE_OBJDUMP=' + llvm_objdump.full_path(),
                            'CMAKE_NM=' + llvm_nm.full_path(),
                            'CMAKE_AR=' + llvm_ar.full_path(),
                            #'KCONFIG_ROOT=' TODO
                            # Disable common warnings that gcc does not emit but clang does. Typedef redefinition will be fixed
                            # in https://github.com/zephyrproject-rtos/zephyr/pull/29359
                            'EXTRA_CFLAGS=-Wno-typedef-redefinition -Wno-unused-command-line-argument @0@'.format(cflags),
                            'CMAKE_EXE_LINKER_FLAGS=@0@'.format(ldflags),
        ]]
    )

    zephyr_targets += target

endif

nop = find_program('true')
all_apps = 'intel-ecfw-zephyr'
run_target(all_apps, command: nop, depends: zephyr_targets)
summary('Build all ARA Zephyr apps', f'meson compile @all_apps@')



