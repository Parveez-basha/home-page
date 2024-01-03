# sampleftp
hai unique process name

import java.io.FileInputStream;
import java.util.Properties;

import com.ibm.connectdirect.ndm.NdmDirect;

public class ConnectDirectFileTransfer {

    public static void main(String[] args) {
        try {
            // Set the NDMAPICFG variable
            String ndmApiConfig = "/ConnectDirect/install/ndm/cfg/cliapi/ndmapi.cfg";
            System.setProperty("NDMAPICFG", ndmApiConfig);

            // Specify the file transfer properties
            Properties transferProps = new Properties();
            transferProps.setProperty("source", "/locallinuxfolder/folder/source_file.txt");
            transferProps.setProperty("target", "//eagle.usaa.com/idtn_test/yourLANshare/destination_file.txt");

            // Perform the file transfer
            NdmDirect.transferFile(transferProps);

            System.out.println("File transferred successfully!");

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}












ACS
Advanced C

de Search

FEEDBACKABOUT
FtpUtil.java
Dark Mode
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
71
72
73
74
75
76
77
78
79
80
81
82
83
84
85
86
87
88
89
90
91
92
93
94
95
96
97
98
99
100
101
102
103
104
105
106
107
108
109
110
111
112
113
114
115
116
117
118
119
120
121
122
123
124
125
126
127
128
129
130
131
132
133
134
135
136
137
138
139
140
141
142
143
144
145
146
147
148
149
150
151
152
153
154
155
156
157
158
159
160
161
162
163
164
165
166
167
168
169
170
171
172
173
174
175
176
177
178
179
180
181
182
183
184
185
186
187
188
189
190
191
192
193
194
195
196
197
198
199
200
201
202
203
204
205
206
207
208
209
210
211
212
213
214
215
216
217
218
219
220
221
222
223
224
225
226
227
228
229
230
231
232
233
234
235
236
237
238
239
240
241
242
243
244
245
246
247
248
249
250
251
252
253
254
255
256
257
258
259
260
261
262
263
264
265
266
267
268
269
270
271
272
273
274
275
276
277
278
279
280
281
282
283
284
285
286
287
288
289
290
291
292
293
294
295
296
297
298
299
300
301
302
303
304
305
306
307
308
309
310
311
312
313
314
315
316
317
318
319
320
321
322
323
324
325
326
327
328
329
330
331
332
333
334
335
336
337
338
339
340
341
342
343
344
345
346
347
348
349
350
351
352
353
354
355
356
357
358
359
360
361
362
363
364
365
366
367
368
369
370
371
372
373
374
375
376
377
378
379
380
381
382
383
384
385
386
387
388
389
390
391
392
393
394
395
396
397
398
399
400
401
402
403
404
405
406
407
408
409
410
411
412
413
414
415
416
417
418
419
420
421
422
423
424
425
426
427
428
429
430
431
432
433
434
435
436
437
438
439
440
441
442
443
444
445
446
447
448
449
450
451
452
453
454
455
456
457
458
459
460
461
462
463
464
465
466
467
468
469
470
471
472
473
474
475
476
477
478
479
480
481
482
483
484
485
486
487
488
489
490
491
492
493
494
495
496
497
498
499
500
501
502
503
504
505
506
507
508
509
510
511
512
513
514
515
516
517
518
519
520
521
522
523
524
525
526
527
528
529
530
531
532
533
534
535
536
537
538
539
540
541
542
543
544
545
546
547
548
549
550
551
552
553
554
555
556
557
558
559
560
561
562
563
564
565
566
567
568
569
570
571
572
573
574
575
576
577
578
579
580
581
582
583
584
585
586
587
588
589
590
591
592
593
594
595
596
597
598
599
600
601
602
603
604
605
606
607
608
609
610
611
612
613
614
615
616
617
618
619
620
621
622
623
624
625
626
627
628
629
630
631
632
633
634
635
636
637
638
639
640
641
642
643
644
645
646
647
648
649
650
651
652
653
654
655
656
657
658
659
660
661
662
663
664
665
666
667
668
669
670
671
672
673
674
675
676
677
678
679
680
681
682
683
684
685
686
687
688
689
690
691
692
693
694
695
696
697
698
699
700
701
702
703
704
705
706
707
708
709
710
711
712
713
714
715
716
717
718
719
720
721
722
723
724
725
726
727
728
729
730
731
732
733
734
735
736
737
738
739
740
741
742
743
744
745
746
747
748
749
750
751
752
753
754
755
756
757
758
759
760
761
762
763
764
765
766
767
768
769
770
771
772
773
774
775
776
777
778
779
780
781
782
783
784
785
786
787
788
789
790
791
792
793
794
795
796
797
798
799
800
801
802
803
804
805
806
807
808
809
810
811
812
813
814
815
816
817
818
819
820
821
822
823
824
825
826
827
828
829
830
831
832
833
834
835
836
837
838
839
840
841
842
843
844
845
846
847
848
849
850
851
852
853
854
855
856
857
858
859
860
861
862
863
864
865
866
867
868
869
870
871
872
873
874
875
876
877
878
879
880
881
882
883
884
885
886
887
888
889
890
891
892
893
894
895
896
897
898
899
900
901
902
903
904
905
906
907
908
909
910
911
912
913
914
915
916
917
918
919
920
921
922
923
924
925
926
927
928
929
930
931
932
933
934
935
936
937
938
939
940
941
942
943
944
945
946
947
948
949
950
951
952
953
954
955
956
957
958
959
960
961
962
963
964
965
966
967
968
969
970
971
972
973
974
975
976
977
978
979
/*******************************************************************************
 * Copyright (C) 2023
 * United Services Automobile Association
 * All Rights Reserved
 * 
 * File: FtpUtil.java
 * ******************************************************************************
 * Target           Chg Date          Name           Description 
 *==========       ==========       ============    ================== 
 *20110415         Jan 06,2011      Kiran Dagara    Initial Creation
 *20111118		   Sep 15,2011	   Pavani Addanki   Modified for REG O Data retention
 *20120127	       Oct 15,2011	   Pavani Addanki   Modified for REG O Data retention
 *20210917		   Aug 27,2021	   Giribabu			Added SAM changes for batch job 1902 & 1903
 *20211008		   Oct 5,2021	   Hidayat			Updated NDM value for batch job 1903
 *xx/xx/2023		07/17/2023		Chris Spangler	RBPRE-2594: Rename file to Fidelity and FDR
 ******************************************************************************/
//Package Section
package com.usaa.bank.insider.util;

import java.io.BufferedReader;
import java.io.BufferedWriter;
//Import Section
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.ArrayList;
import java.util.Calendar;
import java.util.List;

import com.ibm.icu.text.DateFormatSymbols;
import com.usaa.bank.insider.common.InsiderBatchConstants;
import com.usaa.bank.insider.common.InsiderBatchLogger;
import com.usaa.bank.insider.exceptions.PropertiesException;
import com.usaa.bank.insider.exceptions.SystemException;
import com.usaa.bank.insider.validator.FTPBatchValidator;
import com.usaa.ecm.content.ingestion.service.exceptions.ContentIngestionBusinessFault;
import com.usaa.ecm.content.ingestion.service.exceptions.ContentIngestionSystemFault;
import com.usaa.ecm.content.ingestion.service.servicefacade.ws.ContentIngestionService_PortType;
import com.usaa.ecm.content.ingestion.service.valueobjects.ApplicationIdentifierType;
import com.usaa.ecm.content.ingestion.service.valueobjects.AttributesType;
import com.usaa.ecm.content.ingestion.service.valueobjects.ContentAttributesType;
import com.usaa.ecm.content.ingestion.service.valueobjects.ContentGroupIdentifierType;
import com.usaa.ecm.content.ingestion.service.valueobjects.ContentIngestionRequest;
import com.usaa.ecm.content.ingestion.service.valueobjects.ContentIngestionResponse;
import com.usaa.ecm.content.ingestion.service.valueobjects.SourceLocationIdentifierType;
import com.usaa.infrastructure.connectivity.ConnectivityExceptionContainer;
import com.usaa.infrastructure.connectivity.ConnectivityServicesException;
import com.usaa.infrastructure.connectivity.ConnectivityServicesFactory;
import com.usaa.infrastructure.connectivity.IConnectivityServices;
import com.usaa.infrastructure.connectivity.ftp.FTPGetInputBean;
import com.usaa.infrastructure.connectivity.ftp.FTPPutBytesInputBean;
import com.usaa.infrastructure.connectivity.ftp.FTPPutInputBean;
import com.usaa.infrastructure.connectivity.ftp.FTPResultBean;
import com.usaa.infrastructure.constants.IGlobalConstants;
import com.usaa.infrastructure.logging.LogType;
import com.usaa.infrastructure.security.SecurityConfigurationManager; 
import com.usaa.infrastructure.serviceaccount.exception.ServiceAccountException;
import com.usaa.infrastructure.services.locator.ServiceLocator;
import com.usaa.infrastructure.services.locator.ServiceLocatorException;
import com.usaa.infrastructure.util.StringFunctions;

import org.apache.commons.lang3.StringUtils;

import java.net.InetAddress;
import java.net.UnknownHostException;
import java.rmi.RemoteException;
import java.text.SimpleDateFormat;

/**
 * @author bi79394
 *
 * This FtpUtil class holds the method which is used to FTP the file to different systems.
 */
public class FtpUtil
{
	/**
	 * This method sends data from FTP to Host
	 * @param putBytesInputBean
	 * @param interactionName
	 * @param hostAlias
	 * @throws SystemException 
	 */
	private void ftpToHost(FTPPutBytesInputBean putBytesInputBean,String interactionName, String hostAlias) throws SystemException 
	{
		String methodName = this.getClass()+InsiderBatchConstants.COLON+"ftpToHost()";
		InsiderBatchLogger.logMethodEntry(methodName);
		FTPResultBean putOutputBean;
		if (putBytesInputBean != null)
		{		
			try
			{
				IConnectivityServices connServices = ConnectivityServicesFactory
				.getConnectivityServices(
					ConnectivityServicesFactory.DEFAULT_SERVICES, hostAlias);

				putOutputBean = (FTPResultBean) connServices.execute(
						interactionName, putBytesInputBean);
				
				if ((putOutputBean != null)
						&& (putOutputBean.isPositiveCompletion()))
				{
					StringBuffer errMsg = new StringBuffer();
					errMsg.append(methodName).append(InsiderBatchConstants.COLON).append(InsiderBatchConstants.FTP_TO_HOST_SUCCESS).
					append(InsiderBatchConstants.REPLY_CODE).append(putOutputBean.getFTPReplyCode()).
					append(InsiderBatchConstants.REPLY_STRING).append(putOutputBean.getFTPReplyString());

					InsiderBatchLogger.logDebugMessage(errMsg.toString());
				}
				else if(putOutputBean != null)
				{
					StringBuffer errMsg = new StringBuffer();
					errMsg.append(methodName).append(InsiderBatchConstants.COLON).append(InsiderBatchConstants.FTP_TO_HOST_FAIL).
					append(InsiderBatchConstants.REPLY_CODE).append(putOutputBean.getFTPReplyCode()).
					append(InsiderBatchConstants.REPLY_STRING).append(putOutputBean.getFTPReplyString());
					throw new SystemException(
							InsiderBatchConstants.FTP_TO_HOST_FAIL);
				}
			}
			catch (ConnectivityExceptionContainer e)
			{
				StringBuffer errMsg = new StringBuffer();
				errMsg.append(methodName).append(InsiderBatchConstants.COLON).append(InsiderBatchConstants.CONNECTIVITY_EXCEP_CONTAINER);
				throw new SystemException(errMsg.toString(), e);
			}
			catch (ConnectivityServicesException e)
			{
				StringBuffer errMsg = new StringBuffer();
				errMsg.append(methodName).append(InsiderBatchConstants.COLON).append(InsiderBatchConstants.CONNECTIVITY_SERVICES_CONTAINER).
				append(InsiderBatchConstants.CONNECTIVITY_EXCEP_CONTAINER);
				throw new SystemException(errMsg.toString(), e);
			}
			catch (Exception e)
			{
				e.printStackTrace(System.out);
				StringBuffer errMsg = new StringBuffer();
				errMsg.append(methodName).append(InsiderBatchConstants.COLON).append(InsiderBatchConstants.FTP_HOST_EXCEPTION);
				throw new SystemException(errMsg.toString(), e);
			}
		}
		InsiderBatchLogger.logMethodExit(methodName);
	}
	/*
	 * Execute Connect Direct script 
	 * 
	 * @param commandLine
	 * @param pwd
	 * @throws IOException
	 * @throws InterruptedException
	 */
	public static List<String> executeCDScript(String commandLine,String pwd)
			throws IOException, InterruptedException
	{
		final String METHOD_NAME = "executeCDScript() :";
		InsiderBatchLogger.logMethodEntry( METHOD_NAME, InsiderBatchLogger.class);
		Runtime rt = Runtime.getRuntime();
		Process proc = rt.exec(commandLine);
		proc.waitFor();

		// get script feedback
		InputStreamReader scriptFeedback = new InputStreamReader(
				proc.getInputStream());
		BufferedReader reader = new BufferedReader(scriptFeedback);
		List<String> errorMsgList = new ArrayList<String>();
		// Logic for handling script return values
		String output = IGlobalConstants.EMPTY_STRING;
		String line = IGlobalConstants.EMPTY_STRING;
		String returnCode = IGlobalConstants.EMPTY_STRING;
		String message = IGlobalConstants.EMPTY_STRING;
		
		if (reader.readLine() == null)
        {
               errorMsgList.add("Connect Direct is not installed");
        }
		while ((line = reader.readLine()) != null)
		{
			// Connect Direct CL output has shown 3 different formats
			if (line.contains("Return code"))
			{
				// return code value: 0=good. 4,8,16=bad
				// (note the little 'c' on 'code')
				// Return code = 0
				returnCode = StringUtils.substring(line, 36, 38);
				if ("0".equals(returnCode))
				{
					// do stuff (or not) if successful
					return new ArrayList<String>();
				}
				else
				{
					// extract return code and message
					// do stuff to record return code/message
					line = reader.readLine();
					message = StringUtils.substring(line, 36, 44);
					output = "Code: " + returnCode + " msg: " + message + "\n";
					errorMsgList.add("Connect Direct: " + output);
				}
			}
			else if (line.contains("Feedback"))
			{
				// return code value: 0=good. 4,8,16=bad
				returnCode = StringUtils.substring(line, 23, 25);
				message = StringUtils.substring(line, 0, 8);
				output = "Code: " + returnCode + " msg: " + message + "\n";
				errorMsgList.add("Connect Direct: " + output);
			}
			else if (line.contains("**** Error returned..."))
			{
				errorMsgList.add("Connect Direct failed, no further information provided. CLI: "
						+ StringUtils.replace(commandLine, pwd, "********"));
			}
		}
		InsiderBatchLogger.logMethodExit( METHOD_NAME, InsiderBatchLogger.class);
		return errorMsgList;
	}
	
	/**
	 * This method sends data to different systems (FDR,Fidelity,CreditRevue)
	 * by preparing FTPPutBytesInputBean and calls ftpToHost method to ftp the data
	 * to respective systems based on parameter system.
	 * @param inputData
	 * @param system
	 * @throws IOException
	 * @throws SystemException 
	 * @throws PropertiesException 
	 */
	public void sendData(String inputData,String system) throws IOException, SystemException, PropertiesException
	{
		String methodName = FtpUtil.class + InsiderBatchConstants.COLON + "sendData()";
		InsiderBatchLogger.logMethodEntry(methodName);
		
		byte input[] = inputData.getBytes();
		FTPBatchValidator validator = new FTPBatchValidator();
		validator.validate(system, inputData);
		if (!sendDataUsingCyberark(system, inputData)) {
			if (system.equalsIgnoreCase(InsiderBatchConstants.FDR_SYSTEM)) {
				FTPPutBytesInputBean putBytesInputBean = new FTPPutBytesInputBean();
				putBytesInputBean
						.setDestinationFile(BatchProperties.getValue(InsiderBatchConstants.FDR_HOST_IN_DATASET_NAME));
				putBytesInputBean.setWorkingDir(InsiderBatchConstants.HOST_DESTN_PATH);
				putBytesInputBean.setBytes(input);
				putBytesInputBean.setBinaryTransfer(false);
				putBytesInputBean.addSiteCommand(InsiderBatchConstants.SITE_COMMAND + inputData.length());
				ftpToHost(putBytesInputBean, InsiderBatchConstants.FDR_HOST_IN_INTERACTION_NAME,
						BatchProperties.getValue(InsiderBatchConstants.FTP_FDR_HOST_CREDENTIALS_ALIAS));
			} else if (system.equalsIgnoreCase(InsiderBatchConstants.FIDELITY_SYSTEM)) {
				FTPPutBytesInputBean putBytesInputBean = new FTPPutBytesInputBean();
				putBytesInputBean.setDestinationFile(
						BatchProperties.getValue(InsiderBatchConstants.FIDELITY_HOST_IN_DATASET_NAME));
				putBytesInputBean.setWorkingDir(InsiderBatchConstants.HOST_DESTN_PATH);
				putBytesInputBean.setBytes(input);
				putBytesInputBean.setBinaryTransfer(false);
				putBytesInputBean.addSiteCommand(InsiderBatchConstants.SITE_COMMAND + inputData.length());

				ftpToHost(putBytesInputBean, InsiderBatchConstants.FIDELITY_HOST_IN_INTERACTION_NAME,
						BatchProperties.getValue(InsiderBatchConstants.FTP_FIDELITY_HOST_CREDENTIALS_ALIAS));
			} else if (system.equalsIgnoreCase(InsiderBatchConstants.CR_SYSTEM)) {
				FTPPutBytesInputBean putBytesInputBean = new FTPPutBytesInputBean();
				InsiderBatchUtil batchUtil = new InsiderBatchUtil();
				String destFile = BatchProperties.getValue(InsiderBatchConstants.CREDIT_REVUE_HOST_IN_DATASET_NAME)
						+ InsiderBatchConstants.HYPHEN + batchUtil.getCurrentDate()
						+ InsiderBatchConstants.FILE_EXTENSION;
				putBytesInputBean.setDestinationFile(destFile);
				putBytesInputBean.setWorkingDir(BatchProperties.getValue(InsiderBatchConstants.CR_DESTN_PATH));
				putBytesInputBean.setBytes(input);
				putBytesInputBean.setBinaryTransfer(false);

				ftpToHost(putBytesInputBean, InsiderBatchConstants.CR_HOST_IN_INTERACTION_NAME,
						BatchProperties.getValue(InsiderBatchConstants.FTP_CREDIT_REVUE_HOST_CREDENTIALS_ALIAS));
			} else {
				InsiderBatchLogger.logDebugMessage(InsiderBatchConstants.SYSTEM_UNDEFINED);
			}
		} else {
			InsiderBatchLogger.logErrorMessage("File Placed to Host thru Connect direct using cyberark credentials",
					"ERROR_WARNING");
			InsiderBatchLogger.logMethodExit(methodName);
		}
	}	
	
	/**
	 * This method is called while FTP (sending) data to Host/other systems using connect direct
	 * @param system
	 * @param inputData
	 * 
	 */
	public boolean sendDataUsingCyberark(String system,String inputData)
			throws PropertiesException
	{
		String methodName = this.getClass() + InsiderBatchConstants.COLON + "sendDataUsingCyberark()";
		InsiderBatchLogger.logMethodEntry(methodName);
		List<String> messageList = new ArrayList<String>();
		String sNode = IGlobalConstants.EMPTY_STRING;

		/**
		 * Below steps are followed to use connect direct 
		 * 1. check if erdc flag is turned on or not 
		 * 2. If ERDC flag is on, then retrieve cyberark SA 
		 * 3. cyberark SA is successful, prepare the input txt file 
		 * 4. prepare command line 
		 * 5. execute CD script
		 * 6. validate the response
		 */

		// step 1: check if erdc flag is turned on or not
		// Step 2: retrieve cyberark SA
		ERDCUtil erdcUtil = new ERDCUtil();
		FTPPutBytesInputBean putBytesInputBean = new FTPPutBytesInputBean();
		List<String> userDetails = new ArrayList<String>();
		String commandLine = "";
		if (system.equalsIgnoreCase(InsiderBatchConstants.FDR_SYSTEM)) {
			if (!erdcUtil.isCyberArkEnabled(InsiderBatchConstants.CYBERARK_ERDC_FLAG_NAME_1902_BATCH)) {
				return false;
			}
			if(erdcUtil.readAndPushNewFDROutputDataset()) {

				userDetails = cyberArkUserDetails(InsiderBatchConstants.NEW_CYBERARK_ALIAS_FOR_HOST_1902,
						InsiderBatchConstants.NEW_CYBERARK_APP_ID_FOR_HOST_1902);
				putBytesInputBean.setDestinationFile(
						BatchProperties.getValue(InsiderBatchConstants.NEW_FDR_HOST_IN_DATASET_NAME_FOR_CD));
				sNode=BatchProperties.getValue(InsiderBatchConstants.NEW_FTP_HOST_SNODE);
			}
			
			else {
				userDetails = cyberArkUserDetails(InsiderBatchConstants.CYBERARK_ALIAS_FOR_HOST_1902,
						InsiderBatchConstants.CYBERARK_APP_ID_FOR_HOST_1902);
				putBytesInputBean.setDestinationFile(
						BatchProperties.getValue(InsiderBatchConstants.FDR_HOST_IN_DATASET_NAME_FOR_CD));
				sNode = BatchProperties.getValue(InsiderBatchConstants.FTP_HOST_SNODE);
			}
			//Flag on: FDR input file and command line will use rego_file_fdr.txt
			if (erdcUtil.isRenamedRegOSourceFile()) {
				// step 3: prepare the FDR input txt file
				inputDataUpdateToFile(inputData, InsiderBatchConstants.FDR_SOURCE_FILE_PATH);
								// step 4: prepare command line
				commandLine = prepareCommandLine(userDetails, putBytesInputBean, sNode, InsiderBatchConstants.FDR_SOURCE_FILE_PATH);
			}			
		} else if (system.equalsIgnoreCase(InsiderBatchConstants.FIDELITY_SYSTEM)) {
			if (!erdcUtil.isCyberArkEnabled(InsiderBatchConstants.CYBERARK_ERDC_FLAG_NAME_1903_BATCH)) {
				return false;
			}
			userDetails = cyberArkUserDetails(InsiderBatchConstants.CYBERARK_ALIAS_FOR_1903,
					InsiderBatchConstants.CYBERARK_APP_ID_FOR_1903);
			putBytesInputBean.setDestinationFile(
					BatchProperties.getValue(InsiderBatchConstants.FIDELITY_HOST_IN_DATASET_NAME_FOR_CD));
			sNode = BatchProperties.getValue(InsiderBatchConstants.FTP_FIDELITY_SNODE);
			
			//Flag on: Fidelity input file and command line will use rego_file_fidelity.txt
			if (erdcUtil.isRenamedRegOSourceFile()) {
				// step 3: prepare the Fidelity input txt file
				inputDataUpdateToFile(inputData, InsiderBatchConstants.FIDELITY_SOURCE_FILE_PATH);
				
				// step 4: prepare command line
				commandLine = prepareCommandLine(userDetails, putBytesInputBean, sNode, InsiderBatchConstants.FIDELITY_SOURCE_FILE_PATH);
			}			
		} else {
			InsiderBatchLogger.logDebugMessage(InsiderBatchConstants.SYSTEM_UNDEFINED);
			return false;
		}
		// Validating userDetails
		if (userDetails.get(0).equals(IGlobalConstants.EMPTY_STRING)
				&& userDetails.get(1).equals(IGlobalConstants.EMPTY_STRING)) {
			return false;
		}
		
		//Flag off: both FDR and Fidelity will use rego_file.txt
		if (!erdcUtil.isRenamedRegOSourceFile()) {
			// step 3: prepare the input txt file
			inputDataUpdateToFile(inputData, InsiderBatchConstants.SOURCE_FILE_PATH);
			// step 4: prepare command line
			commandLine = prepareCommandLine(userDetails, putBytesInputBean, sNode, InsiderBatchConstants.SOURCE_FILE_PATH);
		}
		
		// step5: execute CD script
		try {
			messageList = executeCDScript(commandLine, userDetails.get(1));
		} catch (Exception e) {
			InsiderBatchLogger.logError(LogType.ERROR_CRITICAL,
					"Unable to process command line arguments required to execute Connect Direct Shell Script::", e);
			return false;
		}

		// step 6: Validate the response
		if (messageList.isEmpty()) {
			InsiderBatchLogger.logErrorMessage("Connect direct Success", LogType.ERROR_WARNING);
			InsiderBatchLogger.logMethodExit(methodName, InsiderBatchLogger.class);
			return true;
		} else {
			InsiderBatchLogger.logErrorMessage("Connect direct Failure - retry with SAU, " + messageList.toString(),
					LogType.ERROR_WARNING);
			InsiderBatchLogger.logMethodExit(methodName, InsiderBatchLogger.class);
			return false;
		}

	}
	
	/**
	 * This method is called to prepare source file date.
	 * @param inputData
	 */
	private void inputDataUpdateToFile(String inputData, String filePath) {
		try {
			BufferedWriter bf=new BufferedWriter(new java.io.FileWriter(filePath));
			bf.write(inputData);
			bf.close();
		} catch (IOException e1) {
			e1.printStackTrace();
		}
	}
	
	/**
	 * Auxiliary method for setting up the command line to send the source file either to FDR or Fidelity
	 * @param userDetails
	 * @param putBytesInputBean
	 * @param sNode
	 * @param sourcePath
	 * @return String commandLine
	 */
	private String prepareCommandLine(List<String> userDetails, FTPPutBytesInputBean putBytesInputBean, 
			String sNode, String sourcePath) {
		return InsiderBatchConstants.PUSH_FILE_USING_CONNECTDIRECT_PATH
				+ InsiderBatchConstants.PUSH_FILE_USING_CONNECTDIRECT_NAME + IGlobalConstants.SPACE_STRING
				+ userDetails.get(0) + IGlobalConstants.SPACE_STRING + userDetails.get(1)
				+ IGlobalConstants.SPACE_STRING + sourcePath + IGlobalConstants.SPACE_STRING
				+ sNode + IGlobalConstants.SPACE_STRING + putBytesInputBean.getDestinationFile()
				+ IGlobalConstants.SPACE_STRING + InsiderBatchConstants.RECORD_LENGTH;
	}
	
	/**
	 * This method is called to retrieve cyberArk SA credentials
	 */
	private List<String> cyberArkUserDetails(String cyberArkAlias, String appId) {

		List<String> userList = new ArrayList<String>();
		
		String username = IGlobalConstants.EMPTY_STRING;
		String pwd = IGlobalConstants.EMPTY_STRING;
		try {
			username = SecurityConfigurationManager.getUserIdForAlias(appId, cyberArkAlias);
			pwd = SecurityConfigurationManager.getPasswordForAlias(appId, cyberArkAlias);
		} catch (ServiceAccountException e) {
			InsiderBatchLogger.logError(LogType.ERROR_CRITICAL,
					"Exception occured while retrieving service account credentials from CyberArk"
							+ " to perform a Connnect Direct file transfer to Host GDG",
					e);
		}

		userList.add(username);
		userList.add(pwd);
		return userList;
	}
	
	/**
	 * This method is called while FTP (receiving) data from Host/other systems.
	 * @param getInputBean
	 * @param interactionName
	 * @param hostAlias
	 * @throws SystemException 
	 */
	public void ftpFromHost(FTPGetInputBean getInputBean,
			String interactionName, String hostAlias) throws SystemException 
	{
		String methodName = this.getClass()+InsiderBatchConstants.COLON+"ftpFromHost()";
		InsiderBatchLogger.logMethodEntry(methodName);		
		
		IConnectivityServices connServices = ConnectivityServicesFactory
				.getConnectivityServices(
						ConnectivityServicesFactory.DEFAULT_SERVICES, hostAlias);
		FTPResultBean getOutputBean = null;
		try
		{
			getOutputBean = (FTPResultBean) connServices.execute(
					interactionName, getInputBean);
			
			if ((getOutputBean != null)
					&& (getOutputBean.isPositiveCompletion()))
			{				
				StringBuffer errMsg = new StringBuffer();
				errMsg.append(methodName).append(InsiderBatchConstants.COLON).append(InsiderBatchConstants.FTP_FROM_HOST_SUCCESS).
				append(InsiderBatchConstants.REPLY_CODE).append(getOutputBean.getFTPReplyCode()).
				append(InsiderBatchConstants.REPLY_STRING).append(getOutputBean.getFTPReplyString());

				InsiderBatchLogger.logDebugMessage(errMsg.toString());
			}
			else if(getOutputBean != null)
			{
				StringBuffer errMsg = new StringBuffer();
				errMsg.append(methodName).append(InsiderBatchConstants.COLON).append(InsiderBatchConstants.FTP_FROM_HOST_FAIL).
				append(InsiderBatchConstants.REPLY_CODE).append(getOutputBean.getFTPReplyCode()).
				append(InsiderBatchConstants.REPLY_STRING).append(getOutputBean.getFTPReplyString());
				throw new SystemException(errMsg.toString());
			}
			
		}
		catch (ConnectivityExceptionContainer e)
		{
			StringBuffer errMsg = new StringBuffer();
			errMsg.append(methodName).append(InsiderBatchConstants.COLON).append(InsiderBatchConstants.CONNECTIVITY_EXCEP_CONTAINER_FTP_HOST);
			throw new SystemException(errMsg.toString(), e);		
		}
		catch (ConnectivityServicesException e)
		{
			StringBuffer errMsg = new StringBuffer();
			errMsg.append(methodName).append(InsiderBatchConstants.COLON).append(InsiderBatchConstants.CONNECTIVITY_SERVICES_CONTAINER_FTP_HOST);
			throw new SystemException(errMsg.toString(), e);			
		}	
		InsiderBatchLogger.logMethodExit(methodName);		
	}
	
	/**
	 * This method is called to map the data from input object to FTP Get Input Bean.
	 * @param system
	 * @param destFile
	 * @param sourceFile
	 * @param destinationPath
	 * @throws SystemException 
	 * @throws PropertiesException 
	 */
	public void receiveData(String system,String destFile,String sourceFile,String destinationPath) throws SystemException, PropertiesException
	{
		String methodName = this.getClass()+InsiderBatchConstants.COLON+"receiveData()";
		InsiderBatchLogger.logMethodEntry(methodName);
		FTPBatchValidator validator = new FTPBatchValidator();
		validator.validateInputs(system,destFile,sourceFile,destinationPath);
		if(system.equalsIgnoreCase(InsiderBatchConstants.FDR_SYSTEM))
		{
			FTPGetInputBean getInputBean = new FTPGetInputBean();
			getInputBean.setDestinationFile(destFile);
			getInputBean.setSourceFile(sourceFile);
			getInputBean.setWorkingDir(destinationPath);			
			getInputBean.setBinaryTransfer(false);
//			getInputBean.addSiteCommand(InsiderBatchConstants.SITE_COMMAND);
			ftpFromHost(getInputBean,InsiderBatchConstants.FDR_HOST_OUT_INTERACTION_NAME, 
					BatchProperties.getValue(InsiderBatchConstants.FTP_FDR_HOST_CREDENTIALS_ALIAS));
		}
		else if(system.equalsIgnoreCase(InsiderBatchConstants.FIDELITY_SYSTEM))
		{
			FTPGetInputBean getInputBean = new FTPGetInputBean();
			getInputBean.setDestinationFile(destFile);
			getInputBean.setSourceFile(sourceFile);
			getInputBean.setWorkingDir(destinationPath);			
			getInputBean.setBinaryTransfer(false);
//			getInputBean.addSiteCommand(InsiderBatchConstants.SITE_COMMAND);			
			ftpFromHost(getInputBean,InsiderBatchConstants.HOST_OUT_INTERACTION_NAME,
					BatchProperties.getValue(InsiderBatchConstants.FTP_HOST_CREDENTIALS_ALIAS));
		}		
		else
		{
			StringBuffer errMsg = new StringBuffer();
			errMsg.append(methodName).append(InsiderBatchConstants.COLON).append(InsiderBatchConstants.SYSTEM_UNDEFINED);
			throw new SystemException(errMsg.toString());			
		}
		InsiderBatchLogger.logMethodExit(methodName);		
	}

	/**
	 * This Method is used to Format any given String. If the length is less than count digit, 
	 * It appends Zero at Left hand side.
	 * @param number
	 * @param count
	 * @return String
	 * 
	 */
	public static String formatString(String number, int count)
	{
		String methodName = FtpUtil.class+InsiderBatchConstants.COLON+"sendFtpReports()";
		InsiderBatchLogger.logMethodEntry(methodName);
		String formattedString = StringFunctions.EMPTY_STRING;
		
		if(!StringFunctions.isNullOrEmpty(number))
		{
			if(number.length() >= count)
			{
				formattedString = number;
			}
			else
			{
				formattedString = StringFunctions.padLeadingZeroes(number, count);
			}
		}
		InsiderBatchLogger.logMethodExit(methodName);
		return formattedString;
	}
	
	/**
	 * This uses the FTPPutInputBean to ftp the reports from one system to other system.. 
	 * This is a method used to ftp the file from websphere location to 
	 * 1. ***FTP*****.USAA.COM
	 * 
	 * @param system
	 * @param destFile
	 * @param sourceFile
	 * @param destinationPath
	 * @throws SystemException
	 */
	public void sendFtpReports(String system, String destFile, String sourceFile, String destinationPath) throws SystemException
	{
		String methodName = this.getClass()+InsiderBatchConstants.COLON+"sendFtpReports()";
		InsiderBatchLogger.logMethodEntry(methodName);
		FTPBatchValidator validator = new FTPBatchValidator();
		validator.validateInputs(system,destFile,sourceFile,destinationPath);
		FTPPutInputBean putInputBean = null;
		if(InsiderBatchConstants.BUSINESS.equalsIgnoreCase(system))
		{
			putInputBean = new FTPPutInputBean();
			putInputBean.setDestinationFile(destFile);
			putInputBean.setSourceFile(sourceFile);
			putInputBean.setWorkingDir(destinationPath);			
			putInputBean.setBinaryTransfer(true);
		}
		else
		{
			StringBuffer errMsg = new StringBuffer();
			errMsg.append(methodName).append(InsiderBatchConstants.COLON).append(InsiderBatchConstants.SYSTEM_UNDEFINED);
			throw new SystemException(errMsg.toString());			
		}
		if (putInputBean != null)
		{		
			try
			{
				IConnectivityServices connServices = ConnectivityServicesFactory
				.getConnectivityServices(
					ConnectivityServicesFactory.DEFAULT_SERVICES, BatchProperties.getValue(InsiderBatchConstants.BUSINESS_HOST_ALIAS));
				
				FTPResultBean putOutputBean;

				putOutputBean = (FTPResultBean) connServices.execute(
						InsiderBatchConstants.BUSINESS_INTERACTION_NAME, putInputBean);
				
				if ((putOutputBean != null)
						&& (putOutputBean.isPositiveCompletion()))
				{
					StringBuffer success = new StringBuffer();
					success.append(methodName).append(InsiderBatchConstants.COLON).append(InsiderBatchConstants.FTP_TO_BUSINESS_SUCCESS).
					append(InsiderBatchConstants.REPLY_CODE).append(putOutputBean.getFTPReplyCode()).
					append(InsiderBatchConstants.REPLY_STRING).append(putOutputBean.getFTPReplyString());

					InsiderBatchLogger.logDebugMessage(success.toString());
				}
				else if(putOutputBean != null)
				{
					StringBuffer errMsg = new StringBuffer();
					errMsg.append(methodName).append(InsiderBatchConstants.COLON).append(InsiderBatchConstants.FTP_TO_BUSINESS_FAIL).
					append(InsiderBatchConstants.REPLY_CODE).append(putOutputBean.getFTPReplyCode()).
					append(InsiderBatchConstants.REPLY_STRING).append(putOutputBean.getFTPReplyString());
					throw new SystemException(errMsg.toString());
				}
			}
			catch (ConnectivityExceptionContainer e)
			{
				StringBuffer errMsg = new StringBuffer();
				errMsg.append(methodName).append(InsiderBatchConstants.COLON).append(InsiderBatchConstants.CONNECTIVITY_EXCEP_CONTAINER).append(e.getMessage());
				throw new SystemException(errMsg.toString(), e);
			}
			catch (ConnectivityServicesException e)
			{
				StringBuffer errMsg = new StringBuffer();
				errMsg.append(methodName).append(InsiderBatchConstants.COLON).append(InsiderBatchConstants.CONNECTIVITY_SERVICES_CONTAINER).append(e.getMessage());
				throw new SystemException(errMsg.toString(), e);
			}
			catch (Exception e)
			{
				StringBuffer errMsg = new StringBuffer();
				errMsg.append(methodName).append(InsiderBatchConstants.COLON).append(InsiderBatchConstants.FTP_HOST_EXCEPTION).append(e.getMessage());
				throw new SystemException(errMsg.toString() , e);
			}
			
		}
		InsiderBatchLogger.logMethodExit(methodName);
	}

	//Starts the changes for DEV039795
	/**
	 * This method invokes the Content Ingestion Web Service to store the
	 * reports in Staff Documentary Repository.
	 * 
	 * @param reportType
	 * @param srcFile
	 * @param destFile
	 * @param reportName
	 * @throws SystemException
	 */
	public void reportsToSDR(String reportType, String srcFile, String destFile, String reportName) throws SystemException 
	{
		String methodName = this.getClass()+InsiderBatchConstants.COLON+"reportsToSDR()";
		InsiderBatchLogger.logMethodEntry(methodName);
		
		ContentIngestionRequest contentIngestionRequest = new ContentIngestionRequest();
		ContentIngestionResponse contentIngestionResponse = new ContentIngestionResponse();
		ContentGroupIdentifierType contentGroup = new ContentGroupIdentifierType();
		ApplicationIdentifierType appIdentifier = new ApplicationIdentifierType();
		SourceLocationIdentifierType sourceLocationIdentifier = new SourceLocationIdentifierType();
		ContentAttributesType contentAttributes = new ContentAttributesType();
		InetAddress address;
		String sourceId=null;
		String path = null;
		String contentKey = null;
		
		contentGroup.setContentGroup(InsiderBatchConstants.CONTENT_GROUP);	
		appIdentifier.setApplicationId(InsiderBatchConstants.APPLICATION_ID);
		sourceLocationIdentifier.setSourceProtocol(InsiderBatchConstants.SOURCE_PROTOCOL);
		sourceLocationIdentifier.setSourceLocation(srcFile);

		try 
		{
			address = InetAddress.getLocalHost();
			sourceId = address.getHostName();
		}
		catch (UnknownHostException e) 
		{
			StringBuffer errMsg = new StringBuffer();
			errMsg.append(methodName).append(InsiderBatchConstants.COLON).append(InsiderBatchConstants.UNKNOWN_HOST_EXCEPTION).append(e.getMessage());
			throw new SystemException(errMsg.toString(), e);			
		}				
		sourceLocationIdentifier.setSourceId(sourceId);
		
		AttributesType document_name = new AttributesType();
		document_name.setName(InsiderBatchConstants.DOCUMENT_NAME);
		document_name.setValue(new String[]{destFile});
		
		AttributesType document_date = new AttributesType();
		document_date.setName(InsiderBatchConstants.DOCUMENT_DATE);
		Calendar calendar = Calendar.getInstance();
		SimpleDateFormat simpleDateFormat = new SimpleDateFormat(InsiderBatchConstants.DATE_FORMAT_MMDDYYYY);				
		String currentDate = simpleDateFormat.format(calendar.getTime());
		document_date.setValue(new String[]{currentDate});
					
		contentAttributes.setMetadataAttributes(new AttributesType[]{document_name, document_date});
		
		AttributesType externalAttributes = new AttributesType();
		externalAttributes.setName(InsiderBatchConstants.TARGET_FOLDER_STRUCTURE);
		
		// gets the target folder structure for all the Reg O reports.
		path = getFolderStructure(reportType,reportName);
		externalAttributes.setValue(new String[]{path});		
		contentAttributes.setExternalAttributes(new AttributesType[]{externalAttributes});
		
		contentIngestionRequest.setContentGroupIdentifier( contentGroup );
		contentIngestionRequest.setApplicationIdentifier(appIdentifier);
		contentIngestionRequest.setSourceLocationIdentifier(sourceLocationIdentifier);
		contentIngestionRequest.setContentAttributes(contentAttributes);
		
		ContentIngestionService_PortType proxy = null;
		ServiceLocator locator = ServiceLocator.getInstance();
		try 
		{
			// Calling the Content Ingestion Web Service
			proxy = (ContentIngestionService_PortType) locator.getService(InsiderBatchConstants.CONTENT_INGESTION_SERVICE_NAME + 
																					InsiderBatchConstants.CONTENT_INGESTION_CLIENT_NAME);
			contentIngestionResponse = proxy.ingestContent(contentIngestionRequest);
			
			if(contentIngestionResponse != null)
			{
				if(contentIngestionResponse.getContentKeyIdentifierType() != null)
				{
					contentKey = contentIngestionResponse.getContentKeyIdentifierType().getContentKey();
				}
			}
			validateContentIngestionResponse(contentKey);
		}
		catch (ServiceLocatorException e) 
		{
			StringBuffer errMsg = new StringBuffer();
			errMsg.append(methodName).append(InsiderBatchConstants.COLON).append(InsiderBatchConstants.SERVICE_LOCATOR_EXCEPTION).append(e.getMessage());
			throw new SystemException(errMsg.toString(), e);			
		}
		catch (RemoteException e) 
		{
			StringBuffer errMsg = new StringBuffer();
			errMsg.append(methodName).append(InsiderBatchConstants.COLON).append(InsiderBatchConstants.REMOTE_EXCEPTION).append(e.getMessage());
			throw new SystemException(errMsg.toString(), e);
		} 
		catch (ContentIngestionSystemFault e) 
		{
			StringBuffer errMsg = new StringBuffer();
			errMsg.append(methodName).append(InsiderBatchConstants.COLON).append(InsiderBatchConstants.CONTENT_INGESTION_SYSTEM_FAULT).append(e.getMessage());
			throw new SystemException(errMsg.toString(), e);
		}
		catch (ContentIngestionBusinessFault e) 
		{
			StringBuffer errMsg = new StringBuffer();
			errMsg.append(methodName).append(InsiderBatchConstants.COLON).append(InsiderBatchConstants.CONTENT_INGESTION_BUSINESS_FAULT).append(e.getMessage());
			throw new SystemException(errMsg.toString(), e);
		}	
		InsiderBatchLogger.logMethodExit(methodName);				
	}
	
	/**
	 * This method is validates the Content Ingestion response
	 * 
	 * @param contentKey
	 * @throws SystemException
	 */
	
	private void validateContentIngestionResponse(String contentKey) throws SystemException 
	{
		String methodName = this.getClass()+InsiderBatchConstants.COLON+"validateContentIngestionResponse()";
		StringBuffer errorMsg = new StringBuffer();
		if(contentKey == null)
		{
			errorMsg.append(InsiderBatchConstants.CONTENT_KEY_NULL).append(methodName);
			throw new SystemException(errorMsg.toString());
		}
		
	}

	/**
	 * This method gets the folder structure for all the Reg O reports
	 * based on the reportType and reportName
	 * 
	 * @param reportType
	 * @param reportName
	 * @return folder structure 
	 * 
	 */
	private String getFolderStructure(String reportType, String reportName) 
	{
		String methodName = this.getClass()+InsiderBatchConstants.COLON+"getFolderStructure()";
		InsiderBatchLogger.logMethodEntry(methodName);
		
		Calendar calendar = Calendar.getInstance();
		SimpleDateFormat simpleDateFormat = new SimpleDateFormat(InsiderBatchConstants.DATE_FORMAT_FOR_FOLDER_STRUCTURE);
		String currentDate = simpleDateFormat.format(calendar.getTime());
		int year = calendar.get(Calendar.YEAR);
		int mon = calendar.get(Calendar.MONTH);
		int currentMonth = mon+1;
		DateFormatSymbols dateFormatSymbols = new DateFormatSymbols();
		String[] month = dateFormatSymbols.getMonths();

		StringBuffer folderPath = new StringBuffer();
		if((InsiderBatchConstants.INSIDER_LIST_REPORT).equalsIgnoreCase(reportName))
		{
			folderPath.append(InsiderBatchConstants.REG_O_REPORTS).append(InsiderBatchConstants.DELIMITER).
			append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(InsiderBatchConstants.INSIDER_LIST).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(reportType).append(InsiderBatchConstants.DELIMITER).
			append(currentMonth).append(InsiderBatchConstants.UNDERSCORE).append(month[mon]).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).append(currentDate);	
		}
		else if((InsiderBatchConstants.ST_REPORT).equalsIgnoreCase(reportName))
		{
			folderPath.append(InsiderBatchConstants.REG_O_REPORTS).append(InsiderBatchConstants.DELIMITER).
			append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(InsiderBatchConstants.ST_REPORT_NAME).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(reportType).append(InsiderBatchConstants.DELIMITER).
			append(currentMonth).append(InsiderBatchConstants.UNDERSCORE).append(month[mon]).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).append(currentDate);	
		}
		else if((InsiderBatchConstants.DAILY_AGGREGATE_RPT).equalsIgnoreCase(reportName))
		{
			folderPath.append(InsiderBatchConstants.REG_O_REPORTS).append(InsiderBatchConstants.DELIMITER).
			append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(InsiderBatchConstants.TOTAL_AGG_REPORT).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(reportType).append(InsiderBatchConstants.DELIMITER).
			append(currentMonth).append(InsiderBatchConstants.UNDERSCORE).append(month[mon]).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).append(currentDate);	
		}
		else if((InsiderBatchConstants.DAILY_THRESHOLD_RPT).equalsIgnoreCase(reportName))
		{
			folderPath.append(InsiderBatchConstants.REG_O_REPORTS).append(InsiderBatchConstants.DELIMITER).
			append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(InsiderBatchConstants.THRESHOLD_REPORT).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(reportType).append(InsiderBatchConstants.DELIMITER).
			append(currentMonth).append(InsiderBatchConstants.UNDERSCORE).append(month[mon]).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).append(currentDate);	
		}
		else if((InsiderBatchConstants.INSIDER_BATCH_EXCEPTION_REPORT).equalsIgnoreCase(reportName))
		{
			folderPath.append(InsiderBatchConstants.REG_O_REPORTS).append(InsiderBatchConstants.DELIMITER).
			append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(InsiderBatchConstants.INSIDER_BATCH_EXCEPTION_REPORT_NAME).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(reportType).append(InsiderBatchConstants.DELIMITER).
			append(currentMonth).append(InsiderBatchConstants.UNDERSCORE).append(month[mon]).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).append(currentDate);	
		}
		else if((InsiderBatchConstants.DAILY_AGGREGATE_BATCH_EXCEPTION_REPORT).equalsIgnoreCase(reportName))
		{
			folderPath.append(InsiderBatchConstants.REG_O_REPORTS).append(InsiderBatchConstants.DELIMITER).
			append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(InsiderBatchConstants.AGGREGATE_BATCH_EXCEPTION_REPORT_NAME).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(reportType).append(InsiderBatchConstants.DELIMITER).
			append(currentMonth).append(InsiderBatchConstants.UNDERSCORE).append(month[mon]).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).append(currentDate);	
		}
		else if((InsiderBatchConstants.MONTHLY_THRESHOLD_RPT).equalsIgnoreCase(reportName))
		{
			folderPath.append(InsiderBatchConstants.REG_O_REPORTS).append(InsiderBatchConstants.DELIMITER).
			append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(InsiderBatchConstants.THRESHOLD_REPORT).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(reportType).append(InsiderBatchConstants.DELIMITER).
			append(currentMonth).append(InsiderBatchConstants.UNDERSCORE).append(month[mon]).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).append(currentDate);	
		}
		else if((InsiderBatchConstants.MONTHLY_AGGREGATE_RPT).equalsIgnoreCase(reportName))
		{
			folderPath.append(InsiderBatchConstants.REG_O_REPORTS).append(InsiderBatchConstants.DELIMITER).
			append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(InsiderBatchConstants.TOTAL_AGG_REPORT).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(reportType).append(InsiderBatchConstants.DELIMITER).
			append(currentMonth).append(InsiderBatchConstants.UNDERSCORE).append(month[mon]).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).append(currentDate);	
		}
		else if((InsiderBatchConstants.MONTHLY_AGGREGATE_BATCH_EXCEPTION_REPORT).equalsIgnoreCase(reportName))
		{
			folderPath.append(InsiderBatchConstants.REG_O_REPORTS).append(InsiderBatchConstants.DELIMITER).
			append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(InsiderBatchConstants.AGGREGATE_BATCH_EXCEPTION_REPORT_NAME).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(reportType).append(InsiderBatchConstants.DELIMITER).
			append(currentMonth).append(InsiderBatchConstants.UNDERSCORE).append(month[mon]).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).append(currentDate);	
		}
		
		// for NSF/OD reports
		else if((InsiderBatchConstants.FSB_DAILY_NSF_REPORT_FILE_NAME).equalsIgnoreCase(reportName))
		{
			folderPath.append(InsiderBatchConstants.REG_O_REPORTS).append(InsiderBatchConstants.DELIMITER).
			append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(InsiderBatchConstants.NSF_REPORT).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(InsiderBatchConstants.FSB_VALUE).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(reportType).append(InsiderBatchConstants.DELIMITER).
			append(currentMonth).append(InsiderBatchConstants.UNDERSCORE).append(month[mon]).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).append(currentDate);	
		}
		else if((InsiderBatchConstants.FSB_DAILY_OD_REPORT_FILE_NAME).equalsIgnoreCase(reportName))
		{
			folderPath.append(InsiderBatchConstants.REG_O_REPORTS).append(InsiderBatchConstants.DELIMITER).
			append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(InsiderBatchConstants.OD_REPORT).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(InsiderBatchConstants.FSB_VALUE).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(reportType).append(InsiderBatchConstants.DELIMITER).
			append(currentMonth).append(InsiderBatchConstants.UNDERSCORE).append(month[mon]).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).append(currentDate);	
		}
		else if((InsiderBatchConstants.FSB_MONTHLY_NSF_REPORT_FILE_NAME).equalsIgnoreCase(reportName))
		{
			folderPath.append(InsiderBatchConstants.REG_O_REPORTS).append(InsiderBatchConstants.DELIMITER).
			append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(InsiderBatchConstants.NSF_REPORT).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(InsiderBatchConstants.FSB_VALUE).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(reportType).append(InsiderBatchConstants.DELIMITER).
			append(currentMonth).append(InsiderBatchConstants.UNDERSCORE).append(month[mon]).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).append(currentDate);	
		}
		else if((InsiderBatchConstants.FSB_MONTHLY_OD_REPORT_FILE_NAME).equalsIgnoreCase(reportName))
		{
			folderPath.append(InsiderBatchConstants.REG_O_REPORTS).append(InsiderBatchConstants.DELIMITER).
			append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(InsiderBatchConstants.OD_REPORT).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(InsiderBatchConstants.FSB_VALUE).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(reportType).append(InsiderBatchConstants.DELIMITER).
			append(currentMonth).append(InsiderBatchConstants.UNDERSCORE).append(month[mon]).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).append(currentDate);	
		}
		else if((InsiderBatchConstants.USB_DAILY_NSF_REPORT_FILE_NAME).equalsIgnoreCase(reportName))
		{
			folderPath.append(InsiderBatchConstants.REG_O_REPORTS).append(InsiderBatchConstants.DELIMITER).
			append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(InsiderBatchConstants.NSF_REPORT).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(InsiderBatchConstants.USB_VALUE).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(reportType).append(InsiderBatchConstants.DELIMITER).
			append(currentMonth).append(InsiderBatchConstants.UNDERSCORE).append(month[mon]).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).append(currentDate);	
		}
		else if((InsiderBatchConstants.USB_DAILY_OD_REPORT_FILE_NAME).equalsIgnoreCase(reportName))
		{
			folderPath.append(InsiderBatchConstants.REG_O_REPORTS).append(InsiderBatchConstants.DELIMITER).
			append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(InsiderBatchConstants.OD_REPORT).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(InsiderBatchConstants.USB_VALUE).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(reportType).append(InsiderBatchConstants.DELIMITER).
			append(currentMonth).append(InsiderBatchConstants.UNDERSCORE).append(month[mon]).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).append(currentDate);	
		}
		else if((InsiderBatchConstants.USB_MONTHLY_NSF_REPORT_FILE_NAME).equalsIgnoreCase(reportName))
		{
			folderPath.append(InsiderBatchConstants.REG_O_REPORTS).append(InsiderBatchConstants.DELIMITER).
			append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(InsiderBatchConstants.NSF_REPORT).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(InsiderBatchConstants.USB_VALUE).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(reportType).append(InsiderBatchConstants.DELIMITER).
			append(currentMonth).append(InsiderBatchConstants.UNDERSCORE).append(month[mon]).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).append(currentDate);	
		}
		else if((InsiderBatchConstants.USB_MONTHLY_OD_REPORT_FILE_NAME).equalsIgnoreCase(reportName))
		{
			folderPath.append(InsiderBatchConstants.REG_O_REPORTS).append(InsiderBatchConstants.DELIMITER).
			append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(InsiderBatchConstants.OD_REPORT).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(InsiderBatchConstants.USB_VALUE).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).
			append(reportType).append(InsiderBatchConstants.DELIMITER).
			append(currentMonth).append(InsiderBatchConstants.UNDERSCORE).append(month[mon]).append(InsiderBatchConstants.SPACE).append(Integer.toString(year)).append(InsiderBatchConstants.DELIMITER).append(currentDate);	
		}
		
		InsiderBatchLogger.logMethodExit(methodName);
		return folderPath.toString();
	}
	//Ends the changes for DEV039795
}
