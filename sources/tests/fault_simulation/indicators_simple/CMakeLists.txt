kbuild_add_module("fsim_test_indicators_simple" "module.c")
kbuild_link_module("fsim_test_indicators_simple" kedr_fault_simulation)

kedr_test_install_module("fsim_test_indicators_simple")