# This file contains the fastlane.tools configuration
# You can find the documentation at https://docs.fastlane.tools
#
# For a list of all available actions, check out
#
#     https://docs.fastlane.tools/actions
#
# For a list of all available plugins, check out
#
#     https://docs.fastlane.tools/plugins/available-plugins
#

# Uncomment the line if you want fastlane to automatically update itself
# update_fastlane




default_platform(:ios)

platform :ios do

  before_all do
    if is_ci?
      ENV["MATCH_KEYCHAIN_NAME"] = "github_actions_keychain"
      ENV["MATCH_KEYCHAIN_PASSWORD"] = ENV["MATCH_KEYCHAIN_NAME"]
    end
  end


  desc "Push a new release build to the App Store"
  lane :release do
    puts "🚀 Release lane about to start 🚀"
    appName = ENV["APP_NAME"]
  	if is_ci?
        puts "CI detected 💻"
        puts "Creating keychain 🔑🔑"
        sendSlackMessage(message:"🚦 #{appName} is about to deploy to testflgiht ✈", success: true)
  		  create_keychain(
          name: ENV["MATCH_KEYCHAIN_NAME"],
          password: ENV["MATCH_KEYCHAIN_PASSWORD"],
          default_keychain: true,
          unlock: true,
          timeout: 3600,
          add_to_search_list: true
        )
        puts "Syncing code signing 📡"
        sync_code_signing(
          readonly: true,
          app_identifier: ENV["BUNDLE_ID"],
          keychain_name: ENV["MATCH_KEYCHAIN_NAME"],
          keychain_password: ENV["MATCH_KEYCHAIN_PASSWORD"]
        )
        sync_code_signing(
          readonly: true,
          app_identifier: ENV["EXTENSION_BUNDLE_ID"],
          keychain_name: ENV["MATCH_KEYCHAIN_NAME"],
          keychain_password: ENV["MATCH_KEYCHAIN_PASSWORD"]
        )
    else
        puts "Human detected 🤓"
        match(type: 'appstore', app_identifier: ENV["BUNDLE_ID"], readonly: true)
        match(type: 'appstore', app_identifier: ENV["EXTENSION_BUNDLE_ID"], readonly: true)
	  end
    increment_build_number(xcodeproj: ENV["PROJECT_FILE_NAME"])
    commit_version_bump(
        force: true
    )

    build_app(workspace: ENV["PROJECT_WORKSPACE_NAME"], scheme: ENV["SCHEME_NAME"])

    upload_to_testflight(
      username: ENV["DEVELOPER_ACCOUNT_EMAIL"],
      app_identifier: ENV["BUNDLE_ID"],
      itc_provider: ENV["ITC_PROVIDER"], # pass a specific value to the iTMSTransporter -itc_provider option
      skip_waiting_for_build_processing: true,
      apple_id: ENV["APPLE_ID"]
    )

    clean_build_artifacts

    build_number = get_build_number(xcodeproj: ENV["PROJECT_FILE_NAME"])
    version = get_version_number(
      xcodeproj: ENV["PROJECT_FILE_NAME"],
      target: ENV["SCHEME_NAME"]
    )

    add_git_tag(tag: "#{version}.#{build_number}")
    push_to_git_remote
    
    sendSlackMessage(message:"#{appName} #{version}.#{build_number} successfully uploaded to testflight!", success: true)

  end


  desc "Prepare the iOS app for dev or build"
  lane :create_certs do
    produce(
        app_identifier: ENV["EXTENSION_BUNDLE_ID"],
        app_name: "OneSignalNotificationServiceExtension",
        skip_itc: true
    )
    match(
        app_identifier: [ENV["BUNDLE_ID"],ENV["EXTENSION_BUNDLE_ID"]],
        type: "development"
    )
    match(
        app_identifier: [ENV["BUNDLE_ID"],ENV["EXTENSION_BUNDLE_ID"]],
        type: "appstore"
    )
  end

  desc "Prepare the iOS app for dev or build"
  lane :install_certs do
    produce(
        app_identifier: ENV["EXTENSION_BUNDLE_ID"],
        app_name: "OneSignalNotificationServiceExtension",
        skip_itc: true
    )
    match(
        app_identifier: [ENV["BUNDLE_ID"],ENV["EXTENSION_BUNDLE_ID"]],
        type: "development",
        readonly: true
    )
    match(
        app_identifier: [ENV["BUNDLE_ID"],ENV["EXTENSION_BUNDLE_ID"]],
        type: "appstore",
        readonly: true
    )
  end



  desc "Testing slack integration"
  lane :test_slack do
    puts "File name "+ ENV["PROJECT_FILE_NAME"]+" target name "+ENV["SCHEME_NAME"]
    build_number = get_build_number(xcodeproj: ENV["PROJECT_FILE_NAME"])
    version = get_version_number(
      xcodeproj: ENV["PROJECT_FILE_NAME"],
      target: ENV["SCHEME_NAME"]
    )
    #build_app(workspace: ENV["PROJECT_WORKSPACE_NAME"], scheme: ENV["SCHEME_NAME"])
    sendSlackMessage(message:"#{ENV["APP_NAME"]} #{version}.#{build_number} fastlane slack test", success: true)
  end



  error do |lane, exception|
    slack(
      message: exception.message,
      success: false
    )
  end
  
  
  lane :sendSlackMessage do |options|
    message = options[:message]
    success = options[:success]
    built_by = "Local Fastlane"
    if is_ci?
      built_by = "Github Actions"
    end
    slack(
        message: message,
        #channel: "#channel",  # Optional, by default will post to the default channel configured for the POST URL.
        success: success,        # Optional, defaults to true.
        payload: {  # Optional, lets you specify any number of your own Slack attachments.
          "Build Date" => Time.new.to_s,
          "Built by" => built_by,
        },
        default_payloads: [:git_branch, :git_author] # Optional, lets you specify a whitelist of default payloads to include. Pass an empty array to suppress all the default payloads.
              # Don't add this key, or pass nil, if you want all the default payloads. The available default payloads are: `lane`, `test_result`, `git_branch`, `git_author`, `last_git_commit`, `last_git_commit_hash`.
    )
  end




end