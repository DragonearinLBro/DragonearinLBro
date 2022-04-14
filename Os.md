- 👋 Hi, I’m @DragonearinLBro
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...

<!---
DragonearinLBro/DragonearinLBro is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
 
#include <base/files/file_path.h>
#include <base/logging.h>
#include <base/strings/string_split.h>
#include <brillo/flag_helper.h>
#include <brillo/process/process.h>
#include <brillo/syslog_logging.h>
#include <chromeos/switches/modemfwd_switches.h>
 
int main(int argc, char** argv) {
DEFINE_bool(prepare_to_flash, false, "Put the modem in flash mode");
DEFINE_string(flash_fw, "", "Flash file(s) containing firmware to the
modem");
DEFINE_bool(get_fw_info, false, "Get custpack information");
DEFINE_bool(reboot, false, "Reboot the modem");
DEFINE_bool(flash_mode_check, false, "Check if the modem is in flash
mode");
DEFINE_string(fw_version, "", "Version number of the firmware to flash");
 
brillo::FlagHelper::Init(argc, argv, "XX helper for modemfwd");
brillo::InitLog(brillo::kLogToSyslog | brillo::kLogToStderrIfTty);
 
int num_opts = FLAGS_prepare_to_flash + !FLAGS_flash_fw.empty() +
FLAGS_get_fw_info + FLAGS_reboot + FLAGS_flash_mode_check;
if (num_opts != 1) {
LOG(ERROR) << "Must supply exactly one supported action";
return EXIT_FAILURE;
}
 
if (FLAGS_prepare_to_flash) {
LOG(INFO) << "Get modem ready for flashing.";
// Code here
return EXIT_SUCCESS;
}
 
if (FLAGS_get_fw_info) {
Version version;
if (!GetVersion(&version))
return EXIT_FAILURE;
 
printf("%s:%s\n", modemfwd::kFwMain, version.main_version.c_str());
printf("%s:%s\n", modemfwd::kFwCarrierUuid,
version.carrier_uuid.c_str());
printf("%s:%s\n", modemfwd::kFwCarrier,
version.carrier_version.c_str());
printf("%s:%s\n", modemfwd::kFwOem, version.oem_version.c_str());
return EXIT_SUCCESS;
}
 
if (FLAGS_flash_mode_check) {
puts(FlashModeCheck() ? "true" : "false");
return EXIT_SUCCESS;
}
 
bool ret = helper::RebootModem();
if (FLAGS_reboot || !ret)
return ret ? EXIT_SUCCESS : EXIT_FAILURE;
 
if (FLAGS_flash_fw.empty())
return EXIT_SUCCESS;
 
base::StringPairs parsed_versions;
std::string main_version;
std::string carrier_version;
std::string oem_version;
if (!FLAGS_fw_version.empty() &&
base::SplitStringIntoKeyValuePairs(FLAGS_fw_version, ':', ',',
&parsed_versions)) {
for (const auto& pair : parsed_versions) {
if (pair.first == modemfwd::kFwMain) {
main_version = pair.second;
DLOG(INFO) << "Flashing Main version: " << main_version;
} else if (pair.first == modemfwd::kFwCarrier) {
carrier_version = pair.second;
DLOG(INFO) << "Flashing Carrier version: " << carrier_version;
} else if (pair.first == modemfwd::kFwOem) {
oem_version = pair.second;
DLOG(INFO) << "Flashing OEM version: " << oem_version;
} else {
DLOG(WARNING) << "Unkown version type '" << pair.first << "' (='"
<< pair.second << "')";
}
}
}
 
// Flash FWs
...
...
}
