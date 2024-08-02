
The IDL file is formed the same as in JAVA. For example, see Adam_hello_world.idl:

```
module Adam_hello_world {
  @topic
  struct Hi {
    string hiWorld;
  };
};
```

There is a difference here, in that we don't need to directly use ACE to generate export files.
We can generate the supporting c++ files directly from the mpc file:

```
project(*idl): dcps {
  // This project ensures the common components get built first.

  TypeSupport_Files {
    Adam_hello_world.idl
  }
  custom_only = 1
}
```

And this can now be built using the mpc build tools as it was in JAVA:
```
%ACE_ROOT%\bin\mwc.pl -type vs2019
```

NOTE: the automatically generated type support files tells MPC to generate message type support files from the idl file defined in the MPC script.

However, this is not the correct time to build it yet, The mpc file is also used to build other necessary files, such as the publisher:

```
project(*idl): dcps {
  // This project ensures the common components get built first.

	TypeSupport_Files {
		Adam_hello_world.idl
	}
	custom_only = 1
}

project(*Publisher): dcpsexe_with_tcp{
	//

	exename = publisher
	after += *idl

	TypeSupport_Files {
		Adam_hello_world.idl
	}	
	
	Source_Files {
		Publisher.cpp
	}
}
```

However, we also need to define the Publisher.cpp file:
```
#include "Adam_hello_worldTypeSupportImpl.h"

#include <dds/DCPS/Marked_Default_Qos.h>
#include <dds/DCPS/Service_Participant.h>
#include <dds/DCPS/WaitSet.h>
#include <dds/DCPS/StaticIncludes.h>
#if OPENDDS_DO_MANUAL_STATIC_INCLUDES
#  include <dds/DCPS/RTPS/RtpsDiscovery.h>
#  include <dds/DCPS/transport/rtps_udp/RtpsUdp.h>
#endif

#include <dds/DdsDcpsInfrastructureC.h>
#include <dds/DdsDcpsPublicationC.h>

#include <ace/Log_Msg.h>

int ACE_TMAIN(int argc, ACE_TCHAR* argv[])
{
  try {
    // Initialize DomainParticipantFactory
    DDS::DomainParticipantFactory_var dpf =
      TheParticipantFactoryWithArgs(argc, argv);

    // Create DomainParticipant
    DDS::DomainParticipant_var participant =
      dpf->create_participant(42,
                              PARTICIPANT_QOS_DEFAULT,
                              0,
                              OpenDDS::DCPS::DEFAULT_STATUS_MASK);

    if (!participant) {
      ACE_ERROR_RETURN((LM_ERROR,
                        ACE_TEXT("ERROR: %N:%l: main() -")
                        ACE_TEXT(" create_participant failed!\n")),
                       1);
    }

    // Register TypeSupport (Adam_hello_world::Hi)
    Adam_hello_world::HiTypeSupport_var ts =
      new Adam_hello_world::HiTypeSupportImpl;

    if (ts->register_type(participant, "") != DDS::RETCODE_OK) {
      ACE_ERROR_RETURN((LM_ERROR,
                        ACE_TEXT("ERROR: %N:%l: main() -")
                        ACE_TEXT(" register_type failed!\n")),
                       1);
    }

    // Create Topic (Hello World Topic)
    CORBA::String_var type_name = ts->get_type_name();
    DDS::Topic_var topic =
      participant->create_topic("Hello World Topic",
                                type_name,
                                TOPIC_QOS_DEFAULT,
                                0,
                                OpenDDS::DCPS::DEFAULT_STATUS_MASK);

    if (!topic) {
      ACE_ERROR_RETURN((LM_ERROR,
                        ACE_TEXT("ERROR: %N:%l: main() -")
                        ACE_TEXT(" create_topic failed!\n")),
                       1);
    }

    // Create Publisher
    DDS::Publisher_var publisher =
      participant->create_publisher(PUBLISHER_QOS_DEFAULT,
                                    0,
                                    OpenDDS::DCPS::DEFAULT_STATUS_MASK);

    if (!publisher) {
      ACE_ERROR_RETURN((LM_ERROR,
                        ACE_TEXT("ERROR: %N:%l: main() -")
                        ACE_TEXT(" create_publisher failed!\n")),
                       1);
    }

    // Create DataWriter
    DDS::DataWriter_var writer =
      publisher->create_datawriter(topic,
                                   DATAWRITER_QOS_DEFAULT,
                                   0,
                                   OpenDDS::DCPS::DEFAULT_STATUS_MASK);

    if (!writer) {
      ACE_ERROR_RETURN((LM_ERROR,
                        ACE_TEXT("ERROR: %N:%l: main() -")
                        ACE_TEXT(" create_datawriter failed!\n")),
                       1);
    }

    Adam_hello_world::HiDataWriter_var hi_writer =
      Adam_hello_world::HiDataWriter::_narrow(writer);

    if (!hi_writer) {
      ACE_ERROR_RETURN((LM_ERROR,
                        ACE_TEXT("ERROR: %N:%l: main() -")
                        ACE_TEXT(" _narrow failed!\n")),
                       1);
    }

    // Block until Subscriber is available
    DDS::StatusCondition_var condition = writer->get_statuscondition();
    condition->set_enabled_statuses(DDS::PUBLICATION_MATCHED_STATUS);

    DDS::WaitSet_var ws = new DDS::WaitSet;
    ws->attach_condition(condition);

    ACE_DEBUG((LM_DEBUG,
               ACE_TEXT("Block until subscriber is available\n")));

    while (true) {
      DDS::PublicationMatchedStatus matches;
      if (writer->get_publication_matched_status(matches) != ::DDS::RETCODE_OK) {
        ACE_ERROR_RETURN((LM_ERROR,
                          ACE_TEXT("ERROR: %N:%l: main() -")
                          ACE_TEXT(" get_publication_matched_status failed!\n")),
                         1);
      }

      if (matches.current_count >= 1) {
        break;
      }

      DDS::ConditionSeq conditions;
      DDS::Duration_t timeout = { 60, 0 };
      if (ws->wait(conditions, timeout) != DDS::RETCODE_OK) {
        ACE_ERROR_RETURN((LM_ERROR,
                          ACE_TEXT("ERROR: %N:%l: main() -")
                          ACE_TEXT(" wait failed!\n")),
                         1);
      }
    }

    ACE_DEBUG((LM_DEBUG,
               ACE_TEXT("Subscriber is available\n")));

    ws->detach_condition(condition);

    // Write samples
    Adam_hello_world::Hi message;
    message.hiWorld = "Hello, OpenDDS!";

    for (int i = 0; i < 10; ++i) {
      DDS::ReturnCode_t error = hi_writer->write(message, DDS::HANDLE_NIL);

      if (error != DDS::RETCODE_OK) {
        ACE_ERROR((LM_ERROR,
                   ACE_TEXT("ERROR: %N:%l: main() -")
                   ACE_TEXT(" write returned %d!\n"), error));
      }
    }

    // Wait for samples to be acknowledged
    DDS::Duration_t timeout = { 30, 0 };
    if (hi_writer->wait_for_acknowledgments(timeout) != DDS::RETCODE_OK) {
      ACE_ERROR_RETURN((LM_ERROR,
                        ACE_TEXT("ERROR: %N:%l: main() -")
                        ACE_TEXT(" wait_for_acknowledgments failed!\n")),
                       1);
    }

    // Clean-up!
    participant->delete_contained_entities();
    dpf->delete_participant(participant);

    TheServiceParticipant->shutdown();

  } catch (const CORBA::Exception& e) {
    e._tao_print_exception("Exception caught in main():");
    return 1;
  }

  return 0;
}
```

The [Publisher.cpp is documented here](obsidian://open?vault=Obsidian&file=DDS%2FSupporting%20notes%2FPublisher.cpp).

The code may now be compiled as before, but this time since the mpc file has been adjusted to account for both the idl file and the publisher all the corresponding code is generated:

```
%ACE_ROOT%\bin\mwc.pl -type vs2019
```

In my current example, this is the x64 Native Tools Command Prompt for VS 2019 output:
```
C:\Users\Adam\Desktop\OpenDDS-DDS-3.28.1\OpenDDS-DDS-3.28.1\DevGuideExamples\DCPS\Adam_hello_world>%ACE_ROOT%\bin\mwc.pl -type vs2019

MPC_ROOT was set to C:\Users\Adam\Desktop\OpenDDS-DDS-3.28.1\OpenDDS-DDS-3.28.1\ACE_wrappers\MPC.
Using .../OpenDDS-DDS-3.28.1/ACE_wrappers/bin/MakeProjectCreator/config/MPC.cfg
Generating 'vs2019' output using default input
Generation Time: 1s
```

we may now launch VS2019 from the command prompt and compile the autogenerated code:
```
C:\Users\Adam\Desktop\OpenDDS-DDS-3.28.1\OpenDDS-DDS-3.28.1\DevGuideExamples\DCPS\Adam_hello_world>devenv C:\Users\Adam\Desktop\OpenDDS-DDS-3.28.1\OpenDDS-DDS-3.28.1\DevGuideExamples\DCPS\Adam_hello_world\Adam_hello_world.sln
```
Building for a x64 system in VS 2019, this is the output:
```
Build started...
1>------ Build started: Project: Adam_Hello_World_Idl, Configuration: Debug x64 ------
1>Invoking ""C:\Users\Adam\Desktop\OpenDDS-DDS-3.28.1\OpenDDS-DDS-3.28.1\ACE_wrappers\bin\tao_idl" and ..\..\..\bin\opendds_idl on Adam_hello_world.idl"
1>tao-idli_dtnB6b.cpp
1>tao-idli_4I6eoi.cpp
1>Invoking ""C:\Users\Adam\Desktop\OpenDDS-DDS-3.28.1\OpenDDS-DDS-3.28.1\ACE_wrappers\bin\tao_idl" on Adam_hello_worldTypeSupport.idl"
1>tao-idli_ae9Uu8.cpp
2>------ Build started: Project: Adam_Hello_World_Publisher, Configuration: Debug x64 ------
2>Invoking ""C:\Users\Adam\Desktop\OpenDDS-DDS-3.28.1\OpenDDS-DDS-3.28.1\ACE_wrappers\bin\tao_idl" and ..\..\..\bin\opendds_idl on Adam_hello_world.idl"
2>tao-idli_6Q2yEM.cpp
2>tao-idli_5dcxvk.cpp
2>Invoking ""C:\Users\Adam\Desktop\OpenDDS-DDS-3.28.1\OpenDDS-DDS-3.28.1\ACE_wrappers\bin\tao_idl" on Adam_hello_worldTypeSupport.idl"
2>tao-idli_5xGfY0.cpp
2>Adam_hello_worldC.cpp
2>Adam_hello_worldS.cpp
2>Adam_hello_worldTypeSupportC.cpp
2>Adam_hello_worldTypeSupportImpl.cpp
2>Adam_hello_worldTypeSupportS.cpp
2>Publisher.cpp
2>Generating Code...
2>Adam_Hello_World_Publisher.vcxproj -> C:\Users\Adam\Desktop\OpenDDS-DDS-3.28.1\OpenDDS-DDS-3.28.1\DevGuideExamples\DCPS\Adam_hello_world\publisher.exe
========== Build: 2 succeeded, 0 failed, 0 up-to-date, 0 skipped ==========
```

However, for the purpose of this example this is still too soon, since we will create a subscriber to accept the 10 published messages from the publisher. 

The Subscriber.cpp [is documented here](obsidian://open?vault=Obsidian&file=DDS%2FSupporting%20notes%2FSubscriber.cpp). There is also a supporting file necessary (both the cpp and header file) called [DataReaderListenerImpl](obsidian://open?vault=Obsidian&file=DDS%2FSupporting%20notes%2FDataReaderListenerImpl). The mpc file must also be updated to include Subscriber.cpp and its dependency on DataReaderListenerImpl.cpp. 

Here is the complete updated mpc file Adam_hello_world.mpc:

```
project(*idl): dcps {
  // This project ensures the common components get built first.

	TypeSupport_Files {
		Adam_hello_world.idl
	}
	custom_only = 1
}

project(*Publisher): dcpsexe_with_tcp{
	//

	exename = publisher
	after += *idl

	TypeSupport_Files {
		Adam_hello_world.idl
	}	
	
	Source_Files {
		Publisher.cpp
	}
}

project(*Subscriber): dcpsexe_with_tcp{
	//

	exename = subscriber
	after += *publisher

	TypeSupport_Files {
		Adam_hello_world.idl
	}	
	
	Source_Files {
		DataReaderListenerImpl.cpp
		Subscriber.cpp
	}
}
```

note that the mpc file defines that the subscriber code should be built after the publisher code, and that the source files required are [DataReaderListenerImpl.cpp](obsidian://open?vault=Obsidian&file=DDS%2FSupporting%20notes%2FDataReaderListenerImpl) and [Subscriber.cpp](obsidian://open?vault=Obsidian&file=DDS%2FSupporting%20notes%2FSubscriber.cpp).

Here is the complete Subscriber.cpp file:
```
#include "DataReaderListenerImpl.h"
#include "Adam_hello_worldTypeSupportImpl.h"

#include <dds/DCPS/Marked_Default_Qos.h>
#include <dds/DCPS/Service_Participant.h>
#include <dds/DCPS/WaitSet.h>
#include <dds/DCPS/StaticIncludes.h>
#if OPENDDS_DO_MANUAL_STATIC_INCLUDES
#  include <dds/DCPS/RTPS/RtpsDiscovery.h>
#  include <dds/DCPS/transport/rtps_udp/RtpsUdp.h>
#endif

#include <dds/DdsDcpsInfrastructureC.h>
#include <dds/DdsDcpsSubscriptionC.h>

#include <ace/Log_Msg.h>

int ACE_TMAIN(int argc, ACE_TCHAR* argv[])
{
  try {
    // Initialize DomainParticipantFactory
    DDS::DomainParticipantFactory_var dpf =
      TheParticipantFactoryWithArgs(argc, argv);

    // Create DomainParticipant
    DDS::DomainParticipant_var participant =
      dpf->create_participant(42,
                              PARTICIPANT_QOS_DEFAULT,
                              0,
                              OpenDDS::DCPS::DEFAULT_STATUS_MASK);

    if (!participant) {
      ACE_ERROR_RETURN((LM_ERROR,
                        ACE_TEXT("ERROR: %N:%l: main() -")
                        ACE_TEXT(" create_participant failed!\n")),
                       1);
    }

    // Register Type (Adam_hello_world::Hi)
    Adam_hello_world::HiTypeSupport_var ts =
      new Adam_hello_world::HiTypeSupportImpl;

    if (ts->register_type(participant, "") != DDS::RETCODE_OK) {
      ACE_ERROR_RETURN((LM_ERROR,
                        ACE_TEXT("ERROR: %N:%l: main() -")
                        ACE_TEXT(" register_type failed!\n")),
                       1);
    }

    // Create Topic (Hello World Topic)
    CORBA::String_var type_name = ts->get_type_name();
    DDS::Topic_var topic =
      participant->create_topic("Hello World Topic",
                                type_name,
                                TOPIC_QOS_DEFAULT,
                                0,
                                OpenDDS::DCPS::DEFAULT_STATUS_MASK);

    if (!topic) {
      ACE_ERROR_RETURN((LM_ERROR,
                        ACE_TEXT("ERROR: %N:%l: main() -")
                        ACE_TEXT(" create_topic failed!\n")),
                       1);
    }

    // Create Subscriber
    DDS::Subscriber_var subscriber =
      participant->create_subscriber(SUBSCRIBER_QOS_DEFAULT,
                                     0,
                                     OpenDDS::DCPS::DEFAULT_STATUS_MASK);

    if (!subscriber) {
      ACE_ERROR_RETURN((LM_ERROR,
                        ACE_TEXT("ERROR: %N:%l: main() -")
                        ACE_TEXT(" create_subscriber failed!\n")),
                       1);
    }

    // Create DataReader
    DDS::DataReaderListener_var listener(new DataReaderListenerImpl);

    DDS::DataReaderQos reader_qos;
    subscriber->get_default_datareader_qos(reader_qos);
    reader_qos.reliability.kind = DDS::RELIABLE_RELIABILITY_QOS;

    DDS::DataReader_var reader =
      subscriber->create_datareader(topic,
                                    reader_qos,
                                    listener,
                                    OpenDDS::DCPS::DEFAULT_STATUS_MASK);

    if (!reader) {
      ACE_ERROR_RETURN((LM_ERROR,
                        ACE_TEXT("ERROR: %N:%l: main() -")
                        ACE_TEXT(" create_datareader failed!\n")),
                       1);
    }

    Adam_hello_world::HiDataReader_var reader_i =
      Adam_hello_world::HiDataReader::_narrow(reader);

    if (!reader_i) {
      ACE_ERROR_RETURN((LM_ERROR,
                        ACE_TEXT("ERROR: %N:%l: main() -")
                        ACE_TEXT(" _narrow failed!\n")),
                       1);
    }

    // Block until Publisher completes
    DDS::StatusCondition_var condition = reader->get_statuscondition();
    condition->set_enabled_statuses(DDS::SUBSCRIPTION_MATCHED_STATUS);

    DDS::WaitSet_var ws = new DDS::WaitSet;
    ws->attach_condition(condition);

    while (true) {
      DDS::SubscriptionMatchedStatus matches;
      if (reader->get_subscription_matched_status(matches) != DDS::RETCODE_OK) {
        ACE_ERROR_RETURN((LM_ERROR,
                          ACE_TEXT("ERROR: %N:%l: main() -")
                          ACE_TEXT(" get_subscription_matched_status failed!\n")),
                         1);
      }

      if (matches.current_count == 0 && matches.total_count > 0) {
        break;
      }

      DDS::ConditionSeq conditions;
      DDS::Duration_t timeout = { 60, 0 };
      if (ws->wait(conditions, timeout) != DDS::RETCODE_OK) {
        ACE_ERROR_RETURN((LM_ERROR,
                          ACE_TEXT("ERROR: %N:%l: main() -")
                          ACE_TEXT(" wait failed!\n")),
                         1);
      }
    }

    ws->detach_condition(condition);

    // Clean-up!
    participant->delete_contained_entities();
    dpf->delete_participant(participant);

    TheServiceParticipant->shutdown();

  } catch (const CORBA::Exception& e) {
    e._tao_print_exception("Exception caught in main():");
    return 1;
  }

  return 0;
}

```

The full breakdown and explanation is documented [here](obsidian://open?vault=Obsidian&file=DDS%2FSupporting%20notes%2FSubscriber.cpp).

Here is the complete DataReaderListenerImpl.cpp file:
```
#include <ace/Log_Msg.h>
#include <ace/OS_NS_stdlib.h>

#include "DataReaderListenerImpl.h"
#include "Adam_hello_worldTypeSupportC.h"
#include "Adam_hello_worldTypeSupportImpl.h"

#include <iostream>

void
DataReaderListenerImpl::on_requested_deadline_missed(
  DDS::DataReader_ptr /*reader*/,
  const DDS::RequestedDeadlineMissedStatus& /*status*/)
{
}

void
DataReaderListenerImpl::on_requested_incompatible_qos(
  DDS::DataReader_ptr /*reader*/,
  const DDS::RequestedIncompatibleQosStatus& /*status*/)
{
}

void
DataReaderListenerImpl::on_sample_rejected(
  DDS::DataReader_ptr /*reader*/,
  const DDS::SampleRejectedStatus& /*status*/)
{
}

void
DataReaderListenerImpl::on_liveliness_changed(
  DDS::DataReader_ptr /*reader*/,
  const DDS::LivelinessChangedStatus& /*status*/)
{
}

void
DataReaderListenerImpl::on_data_available(DDS::DataReader_ptr reader)
{
  Adam_hello_world::HiDataReader_var reader_i =
    Adam_hello_world::HiDataReader::_narrow(reader);

  if (!reader_i) {
    ACE_ERROR((LM_ERROR,
               ACE_TEXT("ERROR: %N:%l: on_data_available() -")
               ACE_TEXT(" _narrow failed!\n")));
    ACE_OS::exit(1);
  }

  Adam_hello_world::Hi message;
  DDS::SampleInfo info;

  const DDS::ReturnCode_t error = reader_i->take_next_sample(message, info);

  if (error == DDS::RETCODE_OK) {
    std::cout << "SampleInfo.sample_rank = " << info.sample_rank << std::endl;
    std::cout << "SampleInfo.instance_state = " << OpenDDS::DCPS::InstanceState::instance_state_mask_string(info.instance_state) << std::endl;

    if (info.valid_data) {
      std::cout << "Message: hiWorld = " << message.hiWorld.in() << std::endl;
    }

  } else {
    ACE_ERROR((LM_ERROR,
               ACE_TEXT("ERROR: %N:%l: on_data_available() -")
               ACE_TEXT(" take_next_sample failed!\n")));
  }
}

void
DataReaderListenerImpl::on_subscription_matched(
  DDS::DataReader_ptr /*reader*/,
  const DDS::SubscriptionMatchedStatus& /*status*/)
{
}

void
DataReaderListenerImpl::on_sample_lost(
  DDS::DataReader_ptr /*reader*/,
  const DDS::SampleLostStatus& /*status*/)
{
}

```

Here is the complete DataReaderListenerImpl.h file:
```
#ifndef DATAREADER_LISTENER_IMPL_H
#define DATAREADER_LISTENER_IMPL_H

#include <ace/Global_Macros.h>

#include <dds/DdsDcpsSubscriptionC.h>
#include <dds/DCPS/LocalObject.h>
#include <dds/DCPS/Definitions.h>

class DataReaderListenerImpl
  : public virtual OpenDDS::DCPS::LocalObject<DDS::DataReaderListener> {
public:
  virtual void on_requested_deadline_missed(
    DDS::DataReader_ptr reader,
    const DDS::RequestedDeadlineMissedStatus& status);

  virtual void on_requested_incompatible_qos(
    DDS::DataReader_ptr reader,
    const DDS::RequestedIncompatibleQosStatus& status);

  virtual void on_sample_rejected(
    DDS::DataReader_ptr reader,
    const DDS::SampleRejectedStatus& status);

  virtual void on_liveliness_changed(
    DDS::DataReader_ptr reader,
    const DDS::LivelinessChangedStatus& status);

  virtual void on_data_available(
    DDS::DataReader_ptr reader);

  virtual void on_subscription_matched(
    DDS::DataReader_ptr reader,
    const DDS::SubscriptionMatchedStatus& status);

  virtual void on_sample_lost(
    DDS::DataReader_ptr reader,
    const DDS::SampleLostStatus& status);
};

#endif /* DATAREADER_LISTENER_IMPL_H */


```

The complete breakdown of the DataReaderListener is found [here](obsidian://open?vault=Obsidian&file=DDS%2FSupporting%20notes%2FDataReaderListenerImpl). 


We are now in the position where we have defined the idl file, the Publisher, the Subscriber (and supporting DataReaderListener) and the mpc file containing the simple build instructions. We are finally in a position to build:
![[cppCompleteFolder.png]]


1. Using x64 Native Tools Command Prompt for VS 2019, navigate to the OpenDDS directory and use the command `setenv` to call the bat file which sets temporary environment variables:
```
C:\Users\Adam\Desktop\OpenDDS-DDS-3.28.1\OpenDDS-DDS-3.28.1>setenv
```

2. Navigate to the directory in which your code exists and run the code which uses the mpc file to autogenerate a lot of code:
```
C:\Users\Adam\Desktop\OpenDDS-DDS-3.28.1\OpenDDS-DDS-3.28.1\DevGuideExamples\DCPS\Adam_hello_world>%ACE_ROOT%\bin\mwc.pl -type vs2019
```
![[cppPopulatedFolder.png]]

3. Use the command prompt window to open Visual Studio 2019 and build the solution as x64
```
C:\Users\Adam\Desktop\OpenDDS-DDS-3.28.1\OpenDDS-DDS-3.28.1\DevGuideExamples\DCPS\Adam_hello_world>devenv C:\Users\Adam\Desktop\OpenDDS-DDS-3.28.1\OpenDDS-DDS-3.28.1\DevGuideExamples\DCPS\Adam_hello_world\Adam_hello_world.sln
```
![[buildSuccessCPP.png]]


Running the example.

This is unintuitive, and a complex process. I certainly must find a way to simplify the deployment.

Three (yes three!!!!) x64 vs2019 command prompts are needed, and the timing is sensitive.

Window 1

1. Set environment variables in the usual way, then navigate to the current working directory.
2. run `%DDS_ROOT%\bin\DCPSInfoRepo -o simple.ior`

This creates a file which calls back to the DCPSInfoRepo exe so that the publisher and subscriber can find the network.

Window 2
1. Set environment variables in the usual way, then navigate to the current working directory.
2. run `subscriber -DCPSInfoRepo file://simple.ior`

Window 3
1. Set environment variables in the usual way, then navigate to the current working directory.
2. run `publisher -DCPSInfoRepo file://simple.ior`





Better deployment: Place a file called rtps.ini in the target directory:
```
[common]
DCPSGlobalTransportConfig=$file
DCPSDefaultDiscovery=DEFAULT_RTPS

[transport/the_rtps_transport]
transport_type=rtps_udp
```

We may now run from just 2 command windows:

Window 1
1. Set environment variables in the usual way, then navigate to the current working directory.
2. run `subscriber -DCPSConfigFile rtps.ini

Window 2
1. Set environment variables in the usual way, then navigate to the current working directory.
2. run `publisher -DCPSConfigFile rtps.ini


Additional Note:

Building a release version of the sln will fail. The solution to this issue is to re clone the tagged version of OpenDDS and continue again from the start, but when the sln is built in VS2019, you must ensure that the build is set to 'release' rather than to 'debug'.



Further notes. I [asked a question on github about deployment](https://github.com/OpenDDS/OpenDDS/discussions/4684).

```
Hello,

This is on windows (x64 target).

I have worked through the 'getting started' guide and have compiled everything needed to run a publisher (as an executable) and a subscriber (as an executable). Both use the same rtps.ini file for discovery.

However, to run each executable I must first launch an x64 native vs19 command prompt, cd to my opendds directory and run setenv.bat to set the environment variables. I can then call the exe from the same window. What would you recommend I do so that I can simply run each executable from a fresh prompt without the requirement to set the environment variables every time?

Ans:
The full development environment is not needed when running applications. Just make sure that the PATH has all the `*.dll` files that the process will need to load. OpenDDS also supports a static library build where all ACE/TAO/OpenDDS code is built in to the executable file.
```



# Deployment update 15/07/24

CMake by default statically builds the files. The workflow I have identified that makes it so I can have a single deployable executable is mental (basically hacks the CMakeList for all of OpenDDS), but this is what works:

1. Use a fresh copy of the latest tagged release of openDDS.
2. Place the finished directory into DevGuideExamples/DCPS (in my case the directory is named Adam_hello_world).
3. Edit the CMakeLists.txt file in OpenDDS root to include this new subdirectory (around line 150):
```
# Examples and Tests
if(BUILD_TESTING)
  set(_OPENDDS_BUILD_TESTS_DEFAULT TRUE)
else()
  set(_OPENDDS_BUILD_TESTS_DEFAULT FALSE)
endif()
set(OPENDDS_BUILD_TESTS ${_OPENDDS_BUILD_TESTS_DEFAULT} CACHE BOOL "Build OpenDDS Tests")
set(OPENDDS_BUILD_EXAMPLES TRUE CACHE BOOL "Build OpenDDS Examples")
if(OPENDDS_BUILD_EXAMPLES OR OPENDDS_BUILD_TESTS)
  enable_testing()
  add_subdirectory(DevGuideExamples/DCPS/Messenger)
  add_subdirectory(DevGuideExamples/DCPS/Adam_hello_world)
endif()
if(OPENDDS_BUILD_TESTS)
  add_subdirectory(tests)
endif()
```

Compile using CMake commands launched in a command prompt scoped to the openDDS directory:
```
cmake -B build -DCMAKE_UNITY_BUILD=TRUE
cmake --build build
```

There will now be generated 2 exes which run standalone, no need to set environment variables. Jesus!