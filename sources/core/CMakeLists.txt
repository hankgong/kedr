set(kmodule_name "kedr")

set(arch_dir "arch/${Kbuild_ARCH}")

file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/${arch_dir}/lib)

check_begin("Checking the ways to create module parameters with callbacks")
# It is simpler for scripting to have one entry in the cache per case.
if(DEFINED MODULE_PARAM_CREATE_USE_OPS_STRUCT OR DEFINED MODULE_PARAM_CREATE_USE_OPS)
	set(found "TRUE")
else()
	set(found)
endif()
if(NOT found)
	check_try()
	kbuild_try_compile(is_module_param_create_use_ops_struct # RESULT_VAR
		"${CMAKE_CURRENT_BINARY_DIR}/module_param_create_use_ops_struct" # bindir
		"module_param_create_use_ops_struct.c" # srcfile
		OUTPUT_VARIABLE out_ops_struct
	)

	if(is_module_param_create_use_ops_struct)
		set(MODULE_PARAM_CREATE_USE_OPS_STRUCT "TRUE" CACHE INTERNAL
			"To create module parameters with callbacks, one needs to use struct ops"
		)
		set(found "TRUE")
	endif()
endif(NOT found)

if(NOT found)
	check_try()
	kbuild_try_compile(is_module_param_create_use_ops # RESULT_VAR
		"${CMAKE_CURRENT_BINARY_DIR}/module_param_create_use_ops" # bindir
		"module_param_create_use_ops.c" # srcfile
		OUTPUT_VARIABLE out_ops
	)

	if(is_module_param_create_use_ops)
		set(MODULE_PARAM_CREATE_USE_OPS "TRUE" CACHE INTERNAL
			"To create module parameters with callbacks, one needs to use ops"
		)
		set(found "TRUE")
	endif()
endif(NOT found)

if(MODULE_PARAM_CREATE_USE_OPS_STRUCT)
	check_end("use callbacks struct")
elseif(MODULE_PARAM_CREATE_USE_OPS)
	check_end("use callbacks themselves")
else(MODULE_PARAM_CREATE_USE_OPS_STRUCT)
	check_end("unknown")
	message(FATAL_ERROR "Unknown way to create module param with callbacks.")
endif(MODULE_PARAM_CREATE_USE_OPS_STRUCT)



kbuild_include_directories(
	"${CMAKE_CURRENT_SOURCE_DIR}/${arch_dir}/include"
	"${CMAKE_CURRENT_BINARY_DIR}/${arch_dir}/lib"
	"${CMAKE_CURRENT_SOURCE_DIR}"
	)

kbuild_add_module(${kmodule_name}
	"kedr_module.c"
    "kedr_base.c"
    "kedr_instrumentor.c"
    "kedr_functions_support.c"
    "kedr_target_detector.c"

	"kedr_base_internal.h"
	"kedr_instrumentor_internal.h"
	"kedr_functions_support_internal.h"
	"kedr_target_detector_internal.h"

    "${arch_dir}/lib/inat.c"
    "${arch_dir}/lib/insn.c"
    "${arch_dir}/lib/inat-tables.h"
    )

add_custom_command(OUTPUT "${CMAKE_CURRENT_BINARY_DIR}/${arch_dir}/lib/inat-tables.h"
			COMMAND LC_ALL=C awk -f "${CMAKE_CURRENT_SOURCE_DIR}/${arch_dir}/tools/gen-insn-attr-x86.awk"
				"${CMAKE_CURRENT_SOURCE_DIR}/${arch_dir}/lib/x86-opcode-map.txt" >
				"${CMAKE_CURRENT_BINARY_DIR}/${arch_dir}/lib/inat-tables.h"
			DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/${arch_dir}/lib/x86-opcode-map.txt"
			)

kedr_install_kmodule(${kmodule_name})
kedr_install_symvers(${kmodule_name})
