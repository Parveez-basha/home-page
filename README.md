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


