using KAutoHelper;
using MessagingToolkit.QRCode.Codec;
using MessagingToolkit.QRCode.Codec.Data;
using OtpNet;
using RestSharp;
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Diagnostics;
using System.Drawing;
using System.IO;
using System.Linq;
using System.Reflection;
using System.Text;
using System.Text.RegularExpressions;
using System.Threading;
using System.Threading.Tasks;
using System.Windows.Forms;

namespace Bat2FA
{
    public partial class Form1 : Form
    {
        public Form1()
        {
            InitializeComponent();
        }

        private void button3_Click(object sender, EventArgs e)
        {
            this.Invoke(new Action(() =>
            {
                OpenFileDialog file = new OpenFileDialog();
                DialogResult result = file.ShowDialog();
                if (result == DialogResult.OK && !string.IsNullOrWhiteSpace(file.FileName))
                {
                    string[] data = File.ReadAllLines(file.FileName);
                    if (data.Length == 0)
                    {
                        MessageBox.Show("KHÔNG CÓ DỮ LIỆU TRONG FILE");
                    }
                    else
                    {
                        foreach (var t in data)
                        {
                            try
                            {
                                string[] t1 = t.ToString().Split('|');
                                if (t1.Length == 3)
                                {
                                    dataGridView1.Rows.Add(true, dataGridView1.Rows.Count + 1, t1[0], t1[1], t1[2], "", "", "");
                                }
                                else if (t1.Length == 4)
                                {
                                    dataGridView1.Rows.Add(true, dataGridView1.Rows.Count + 1, t1[0], t1[1], t1[2], t1[3], "", "");
                                }
                                else if (t1.Length == 5)
                                {
                                    dataGridView1.Rows.Add(true, dataGridView1.Rows.Count + 1, t1[0], t1[1], t1[2], t1[3], t[4], "");
                                }
                                else
                                {
                                    dataGridView1.Rows.Add(true, dataGridView1.Rows.Count + 1, t1[0], t1[1], "", "", "", "");
                                }
                            }
                            catch { }
                        }
                        SaveData();
                        MessageBox.Show("XONG");

                    }

                }
            }));
        }

        private void button2_Click(object sender, EventArgs e)
        {
            this.Invoke(new Action(() =>
            {
                string[] data = Clipboard.GetText().Split(new string[] { Environment.NewLine }, StringSplitOptions.RemoveEmptyEntries).ToArray();
                if (data.Length == 0)
                {
                    MessageBox.Show("CHƯA COPY DỮ LIỆU");
                }
                else
                {
                    foreach (var t in data)
                    {
                        try
                        {
                            dataGridView1.Rows.Add(true, dataGridView1.Rows.Count + 1, t.ToString(), "", "");
                        }
                        catch { }
                    }
                    //SaveData();
                    MessageBox.Show("XONG");
                }
            }));
        }
        public void SaveData()
        {
            string dat = "";
            foreach (DataGridViewRow row in dataGridView1.Rows)
            {
                string d = row.Cells["uid1"].Value.ToString() + "|" +
                    row.Cells["pass1"].Value.ToString() + "|" +
                    row.Cells["ma2fa1"].Value.ToString() + "|" +
                    row.Cells["cookie1"].Value.ToString() + "|" +
                    row.Cells["token1"].Value.ToString() + "\r\n";
                dat += d;
            }
            Invoke(new Action(() =>
            {
                File.WriteAllText("SAVEDATA.txt", dat);
            }));

        }
        private void button4_Click(object sender, EventArgs e)
        {
            Process.Start("explorer.exe", AppDomain.CurrentDomain.BaseDirectory + "LDPath.txt");
        }

        private void cHỌNTẤTCẢToolStripMenuItem_Click(object sender, EventArgs e)
        {
            foreach (DataGridViewRow row in dataGridView1.Rows)
                row.Cells[0].Value = true;
        }

        private void cHỌNBÔIĐENToolStripMenuItem_Click(object sender, EventArgs e)
        {
            foreach (DataGridViewRow row in dataGridView1.SelectedRows)
                row.Cells[0].Value = true;
        }

        private void bỎCHỌNTẤTCẢToolStripMenuItem_Click(object sender, EventArgs e)
        {
            foreach (DataGridViewRow row in dataGridView1.Rows)
                row.Cells[0].Value = false;
        }

        private void bỎHÀNGBÔIĐENToolStripMenuItem_Click(object sender, EventArgs e)
        {
            foreach (DataGridViewRow row in dataGridView1.SelectedRows)
                row.Cells[0].Value = false;
        }

        private void button5_Click(object sender, EventArgs e)
        {
            Process.Start("explorer.exe", AppDomain.CurrentDomain.BaseDirectory + "Proxy.txt");
        }
        public bool ChangeCollegeProxy(string proxy, string deviceID, int i, int indexLD)
        {
            Trangthai(i, "Add proxy");
            string px = "";
            string port = "";
            string user = "";
            string pass = "";
            if (proxy.Contains("|"))
            {
                px = proxy.Split('|')[0];
                port = proxy.Split('|')[1];
                if (proxy.Split('|').Length == 4)
                {
                    user = proxy.Split('|')[2];
                    pass = proxy.Split('|')[3];
                }

            }
            else if (proxy.Contains(":"))
            {
                px = proxy.Split(':')[0];
                port = proxy.Split(':')[1];
                if (proxy.Split(':').Length == 4)
                {
                    user = proxy.Split(':')[2];
                    pass = proxy.Split(':')[3];
                }
            }

            Thread.Sleep(500);
            string noidung;
            while (true)
            {

                if (Bat2FA.AdbHelper.Checkpackage(LDPath, Encoding.UTF8.GetString(Convert.FromBase64String("YWRiIC1zIA==")) + deviceID + " shell pm list packages", "com.cell47.College_Proxy"))
                {
                    Bat2FA.AdbHelper.ClearDataApp(LDPath, deviceID, "com.cell47.College_Proxy");
                    Thread.Sleep(1500);
                zz:
                    Bat2FA.AdbHelper.OpenApp(LDPath, indexLD, "com.cell47.College_Proxy");
                    Thread.Sleep(6000);
                    if (!Bat2FA.AdbHelper.CheckActivity(deviceID, LDPath).Contains("com.cell47.College_Proxy"))
                    {
                        goto zz;
                    }
                    int k = 0;

                    while (true)
                    {
                        //Bitmap screen = Bat2FA.AdbHelper.capturescreen(deviceID,LDPath);
                        noidung = Bat2FA.AdbHelper.dump(deviceID, LDPath);
                        if (Bat2FA.AdbHelper.getbounds(noidung, deviceID, "Proxy IP", false, LDPath))

                        //if (Bat2FA.AdbHelper.CheckExistImage(LDPath,false, deviceID, "start.png", screen,pathimage))
                        {
                        lai:
                            Thread.Sleep(500);
                            Bat2FA.AdbHelper.getbounds2(noidung, deviceID, "textView_address", true, LDPath);
                            //Bat2FA.AdbHelper.tap(LDPath,deviceID, "280", "90");
                            Thread.Sleep(500);
                            Bat2FA.AdbHelper.inputtext(LDPath, deviceID, px);
                            Thread.Sleep(500);
                            //Bat2FA.AdbHelper.tap(LDPath,deviceID, "280", "130");
                            Bat2FA.AdbHelper.getbounds2(noidung, deviceID, "textView_port", true, LDPath);
                            Thread.Sleep(500);
                            Bat2FA.AdbHelper.inputtext(LDPath, deviceID, port);
                            Thread.Sleep(500);
                            //Bat2FA.AdbHelper.tap(LDPath,deviceID, "280", "160");
                            Bat2FA.AdbHelper.getbounds2(noidung, deviceID, "textView_username", true, LDPath);
                            Thread.Sleep(500);
                            Bat2FA.AdbHelper.inputtext(LDPath, deviceID, user);
                            Thread.Sleep(500);
                            //Bat2FA.AdbHelper.tap(LDPath,deviceID, "280", "200");
                            Bat2FA.AdbHelper.getbounds2(noidung, deviceID, "textView_password", true, LDPath);
                            Thread.Sleep(500);
                            Bat2FA.AdbHelper.inputtext(LDPath, deviceID, pass);
                            Thread.Sleep(500);
                            Bat2FA.AdbHelper.getbounds(noidung, deviceID, "proxy_start_button", true, LDPath);
                            Thread.Sleep(1500);

                            while (!isstop)
                            {
                                noidung = Bat2FA.AdbHelper.dump(deviceID, LDPath);
                                if (!Bat2FA.AdbHelper.getbounds(noidung, deviceID, "proxy_start_button", true, LDPath))
                                {
                                    break;
                                }
                            }
                            int ks = 0;
                            while (!isstop)
                            {
                                noidung = Bat2FA.AdbHelper.dump(deviceID, LDPath);
                                if (Bat2FA.AdbHelper.getbounds(noidung, deviceID, "Please enter a valid", false, LDPath))
                                {
                                    Trangthai(i, "Lỗi IP");
                                    return false;
                                }
                                else if (Bat2FA.AdbHelper.getbounds(noidung, deviceID, "\"OK\"", true, LDPath))
                                {
                                    Trangthai(i, "OK");
                                    break;
                                }
                                else if (Bat2FA.AdbHelper.getbounds(noidung, deviceID, "\"STOP PROXY", false, LDPath))
                                {
                                    Trangthai(i, "OK");
                                    break;
                                }
                                else if (ks >= 2)
                                {
                                    break;
                                }
                                ks++;
                            }
                            Trangthai(i, "Set Proxy Xong");
                            Bat2FA.AdbHelper.Back(LDPath, deviceID, "3");
                            Thread.Sleep(2000);
                            break;
                        }
                        // noidung = Bat2FA.AdbHelper.dump(deviceID, LDPath);
                        //if (Bat2FA.AdbHelper.getbounds(noidung, deviceID, "\"WAIT\"", true, LDPath)|| Bat2FA.AdbHelper.getbounds(noidung, deviceID, "\"OK\"", true, LDPath))
                        //{ }
                        else if (!Bat2FA.AdbHelper.CheckActivity(deviceID, LDPath).Contains("com.cell47.College_Proxy"))
                        {
                            goto zz;
                        }
                        else if (k >= 15)
                        {
                            return false;
                        }
                        else
                        {
                            k++;

                        }

                    }
                    break;

                }
                else
                {
                    Trangthai(i, "Chưa Cài App College Proxy");
                    return false;
                }

            }
            return true;
        }
        string LDPath = "";
        bool isstop;
        Queue<DataGridViewRow> querow = new Queue<DataGridViewRow>();
        List<string> listproxy = new List<string>();
        List<Thread> listthread = new List<Thread>();
        public List<string> listld = new List<string>();

        private void btn_start_Click(object sender, EventArgs e)
        {
            //Thread.Sleep(4000);

            if (Check())
            {
                querow.Clear();
                foreach (DataGridViewRow row in dataGridView1.Rows)
                {
                    if (row.Cells[0].Value.Equals(true))
                    {
                        querow.Enqueue(row);
                    }
                }
                if (querow.Count == 0)
                {
                    MessageBox.Show("CHƯA CHỌN ACCOUNT !");
                    return;
                }
                waitok = 0;
                nhanf5 = false;
                nextluot = false;
                listthread.Clear();
                listindex.Clear();
                isstop = false;
                btn_start.Enabled = false;
                btn_start.BackColor = Color.Gray;
                btn_stop.Enabled = true;
                btn_stop.BackColor = Color.Maroon;
                listdcomuse.Clear();
                Thread tong = new Thread(() =>
                {
                    RunNone();
                    foreach (Thread t in listthread)
                    {
                        t.Join();
                    }
                    Invoke(new Action(() =>
                    {
                        btn_start.Enabled = true;
                        btn_start.BackColor = Color.Green;
                        btn_stop.Enabled = false;
                        btn_stop.BackColor = Color.Gray;
                    }));
                    MessageBox.Show("XONG");
                });
                tong.Start();
                tong.IsBackground = true;
            }
        }
        public void Resetdcom(string proxy, int i)
        {
        s:
            var client = new RestClient("http://192.168.100.111:22222/api/v1/reset?proxy=" + proxy);
            client.Timeout = -1;
            var request = new RestRequest(Method.GET);
            IRestResponse response = client.Execute(request);
            string rs = response.Content;
            if (rs.Contains("Success"))
            {
                Trangthai(i, "Reset OK: " + rs);
                Thread.Sleep(3000);
            }
            else
            {
                Trangthai(i, "Reset Lỗi");
                Thread.Sleep(3000);
                goto s;
            }
        }
        public void RunNone()
        {
            while (querow.Any() && !isstop)
            {
                listthread = new List<Thread>();
                demlogin = 0;
                Queue<string> quelddv = new Queue<string>();
                foreach (var ld in queld)
                {
                    quelddv.Enqueue(ld.ToString());
                }
                while (quelddv.Any())
                {
                    if (!querow.Any()) { break; }
                    Thread t = new Thread(() =>
                    {
                        try
                        {
                            string dv = quelddv.Dequeue();
                            Mainaction(dv);
                            Thread.Sleep(1000);
                        }
                        catch (Exception ex)
                        {
                        }
                    });
                    t.Start();
                    t.IsBackground = true;
                    listthread.Add(t);
                    Thread.Sleep(500);
                    Thread.Sleep(Convert.ToInt32(num_delay.Value) * 1000);
                }
                foreach (Thread t1 in listthread)
                {
                    t1.Join();
                }
            }
        }
        List<int> listindex = new List<int>();
        Object getid = new Object();
        public int GetIndex()
        {
            lock (getid)
            {
                int i = 1;
                while (!isstop)
                {
                    if (listindex.Contains(i))
                    {
                        i++;
                    }
                    else
                    {
                        if (listld.Contains(i.ToString()))
                        {
                            listindex.Add(i);
                            return i;
                        }
                        else
                        {
                            i++;
                        }
                    }
                }
                return i;
            }
        }



        public int waitok = 0;
        int demlogin = 0;
        bool nhanf5 = false;
        //MAIN ACTION
        Object ob = new Object();
        public void Mainaction(string deviceID)
        {
            Random rd = new Random();
        geti:
            int i = 0;
            lock (ob)
            {
                i = querow.Dequeue().Index;
            }

            dataGridView1.Rows[i].DefaultCellStyle.ForeColor = Color.Blue;
            string proxy = ":0";
            if (ra_proxy.Checked || ra_dcom.Checked)
            {
                proxy = dataGridView1.Rows[i].Cells["proxy1"].Value.ToString();
                if (ra_dcom.Checked)
                {
                dc:
                    try
                    {
                        if (listdcomuse.Contains(proxy) && listdcomdone.Contains(proxy))
                        {
                            Trangthai(i, "Reset Dcom");
                            Thread.Sleep(2000);
                            Resetdcom(proxy, i);
                            int delays = Convert.ToInt32(num_timedoi.Value);
                            listdcomdone.Remove(proxy);
                            while (delays > 0)
                            {
                                Trangthai(i, "Đợi " + delays + " giây");
                                delays--;
                                Thread.Sleep(1000);
                            }
                        }
                        else if (listdcomuse.Contains(proxy) && !listdcomdone.Contains(proxy))
                        {
                            while (!listdcomdone.Contains(proxy))
                            {
                                Trangthai(i, "Đợi Proxy hoàn thành");
                                Thread.Sleep(1000);
                            }
                            goto dc;
                        }
                        else if (!listdcomuse.Contains(proxy))
                        {
                            listdcomuse.Add(proxy);
                        }
                    }
                    catch
                    {
                        goto dc;
                    }
                }
            }

            int ID2 = (index * 2) + 5555;
            deviceID = "127.0.0.1:" + ID2.ToString();

        reset:
            if (!Bat2FA.AdbHelper.CheckRunning(index.ToString(), LDPath))
            {
                Trangthai(i, "Mở LDPlayer " + index);
                string size = "480,320,120";
                Bat2FA.AdbHelper.Runcmd2(LDPath, "ldconsole.exe modify --index " + index + " --resolution " + size + " --cpu 4 --memory 4096");
                Thread.Sleep(500);
                Bat2FA.AdbHelper.Runcmd2(LDPath, "ldconsole.exe launch --index " + index);
                Thread.Sleep(10000);
                if (!Bat2FA.AdbHelper.CheckRunning(index.ToString(), LDPath))
                {
                    goto reset;
                }

                Trangthai(i, "Mở LD Thành công");
            }
            int ch = 0;
            int tick = Environment.TickCount;
            while (!isstop)
            {
                string get = Bat2FA.AdbHelper.CheckActivity(deviceID, LDPath);
                Trangthai(i, get);
                if (get.Contains(".Launcher"))
                {
                    Thread.Sleep(1000);
                    break;
                }

                Thread.Sleep(1000);
                Bat2FA.AdbHelper.Getstring(LDPath, "adb connect " + deviceID);
                Thread.Sleep(2000);
                string dv = Bat2FA.AdbHelper.Getstring(LDPath, "adb devices");
                if (dv.Replace(" ", "").Contains(deviceID + "offline"))
                {
                    if (ch >= 3)
                    {
                        AdbHelper.QuitLD(LDPath, index);
                        Thread.Sleep(2000);
                        goto reset;
                    }
                    ch++;
                }
                if (Environment.TickCount - tick > 30000)
                {
                    AdbHelper.QuitLD(LDPath, index);
                    Thread.Sleep(2000);
                    goto reset;
                }
            }
            Bat2FA.AdbHelper.sort(LDPath);
            Bat2FA.AdbHelper.Runcmd(LDPath, deviceID, "shell settings put global heads_up_notifications_enable 0");
            if (proxy.Split(':').Length >= 3)
            {
                if (!ChangeCollegeProxy(proxy, deviceID, i, index))
                {
                    return;
                }

            }
            else
            {
                Bat2FA.AdbHelper.AddProxy(LDPath, deviceID, proxy);
            }

            Thread.Sleep(100);
            Trangthai(i, "Clear App");
            Trangthai(i, "Mở LDPlayer " + index);
             string size = "480,480,120";
             Bat2FA.AdbHelper.Runcmd(LDPath,deviceID, "ldconsole.exe modify --index --resolution " + size + " --cpu 4 --memory 4096");
              Thread.Sleep(5000);


            string packagename = "air.com.coalaa.itexasvn";
           // Bat2FA.AdbHelper.Runcmd(LDPath, deviceID, "shell rm -r /data/data/air.com.coalaa.itexasvn/shared_prefs");
            //Bat2FA.AdbHelper.ClearDataApp(LDPath, deviceID, packagename);
            Thread.Sleep(100);
            waitok++;
            Login(i, deviceID, index, packagename);
            // demlogin++;
            string tt = dataGridView1.Rows[i].Cells["trangthai1"].Value.ToString();
            //while (demlogin < queld.Count)
            //{
            //    Trangthai(i, "Đợi Luồng cuối cùng Login Xong");
            //    Thread.Sleep(100);
            //}
            //Thread.Sleep(1000);
            //lock(lockf5)
            //{
            //    if(!nhanf5)
            //    {
            //        nhanf5 = true;
            //        Trangthai(i, "Nhấn F5");
            //        Process[] process = Process.GetProcessesByName("xClickerNew");
            //        foreach (var item in process)
            //        {
            //            IntPtr handle = item.MainWindowHandle;
            //            AutoControl.SendKeyBoardPress(handle, VKeys.VK_F5);
            //        }
            //        //SendKeys.Send("{F5}");
            //    }    
            //}    
            int k1 = 0;
            //while (true)
            //{
            //    Bitmap screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);
            //    if (Bat2FA.AdbHelper.CheckExistImage(LDPath, true, deviceID, "diem.png", screen, pathimage))
            //    {
            //        break;
            //    }
            //    if (k1 >= 20)
            //    {
            //        Trangthai(i, "Lỗi không tìm được NEW");
            //        Invoke(new Action(() =>
            //        {
            //            File.AppendAllText("LoiDiemDanh.txt", dataGridView1.Rows[i].Cells["uid1"].Value.ToString() + "\r\n");
            //        }));
            //        break;
            //    }
            //    else if (Bat2FA.AdbHelper.CheckExistImage(LDPath, true, deviceID, "thuong.png", screen, pathimage))
            //    {

            //    }
            //    k1++;
            //    Thread.Sleep(2000);
            //}
           // int delay1 = Convert.ToInt32(num_doilogin.Value);

          //  while (delay1 > 0)
           // {
            //    Trangthai(i, tt + " Đợi : " + delay1 + " giây");
           //     Thread.Sleep(1000);
           //     delay1--;
           // }

            k1 = 0;
            
                    
                    
                        //if (Bat2FA.AdbHelper.CheckExistImage(LDPath, true, deviceID, "x.png", screen, pathimage))
                        //{
                        //    Thread.Sleep(1000);
                        //}
                        //Trangthai(i, "Tìm New");
                        //Bat2FA.AdbHelper.swipe(LDPath, deviceID, 20, 150, 80, 150);
                        //Thread.Sleep(1000);
                        //Bat2FA.AdbHelper.swipe(LDPath, deviceID, 20, 150, 80, 150);
                        //Thread.Sleep(1000);
                        //Bat2FA.AdbHelper.swipe(LDPath, deviceID, 80, 150, 20, 150);
                        //Thread.Sleep(1000);
                        //Bat2FA.AdbHelper.tap(LDPath, deviceID, "50", "150");
                  
            //int delay = Convert.ToInt32(doi_auto.Value);
            

            //while (delay > 0 && !nextluot && !outld)
            //{
             //   Trangthai(i, tt + " Đợi : " + delay + " giây");
             //   Thread.Sleep(1000);
              //  delay--;
            //}
            //Trangthai(i, "Check trạng thái");

            //string pathimage1 = AppDomain.CurrentDomain.BaseDirectory + @"IMAGE\CHECK";
            //if (Bat2FA.AdbHelper.CheckExistImageFolder(LDPath, false, deviceID, screen, pathimage1))
            //{
            //    Trangthai(i, "Complete");
            //    tt = "Complete | " + tt;
            //}
            //else
            //{
            //    Trangthai(i, "Failed");
            //    tt = "Failed | " + tt;
            //}
            if (ra_dcom.Checked)
            {
                listdcomdone.Add(proxy);
            }
            //waitok--;
            //while (waitok > 0)
            //{
            //    Trangthai(i, "Đợi các luồng hoàn thành");
            //    Thread.Sleep(1);
            //}
            Thread.Sleep(1000);
            Trangthai(i, tt);
            demlogin = 0;
            waitok = 0;
            nextluot = false;
            outld = false;
            nhanf5 = false;
            Thread.Sleep(2000);
        //SaveData();
        x:
            //listindex.Remove(index);
            if (!querow.Any())
            {
                //Bat2FA.AdbHelper.QuitLD(LDPath, index);
                return;
            }
            else
            {
                goto geti;
            }
        }
        object lockf5 = new object();
        string pathimage = AppDomain.CurrentDomain.BaseDirectory + @"IMAGE\";
        public List<string> listdcomuse = new List<string>();
        public List<string> listdcomdone = new List<string>();
        public bool nextluot = false;
       
        public void Login(int i, string deviceID, int index, string packagename)
        {
            int demloi = 0;
        xong:
            if (nextluot) return;
            if (demloi >= 1)
            {
                Trangthai(i, "xong");
                Invoke(new Action(() =>
                {
                    File.AppendAllText("Loi.txt", dataGridView1.Rows[i].Cells["uid1"].Value.ToString() + "\r\n");
                }));
                dataGridView1.Rows[i].DefaultCellStyle.BackColor = Color.Green;
                return;
            }
        loi:
            if (nextluot) return;
            if (demloi >= 1)
            {
                Trangthai(i, "Lỗi Login");
                Invoke(new Action(() =>
                {
                    File.AppendAllText("Loi.txt", dataGridView1.Rows[i].Cells["uid1"].Value.ToString() + "\r\n");
                }));
                dataGridView1.Rows[i].DefaultCellStyle.BackColor = Color.Red;
                return;
            }
            demloi++;
            Trangthai(i, "Mở App");
            Thread.Sleep(500);
            Bat2FA.AdbHelper.tap(LDPath, deviceID, "239", "239");
            Thread.Sleep(100);
            Bat2FA.AdbHelper.tap(LDPath, deviceID, "239", "239");
            Thread.Sleep(100);
            Bat2FA.AdbHelper.tap(LDPath, deviceID, "239", "239");
            Thread.Sleep(100);
            Bat2FA.AdbHelper.tap(LDPath, deviceID, "397", "50");
            Thread.Sleep(100);
            Bat2FA.AdbHelper.tap(LDPath, deviceID, "397", "50");
            Thread.Sleep(100);
            Bat2FA.AdbHelper.tap(LDPath, deviceID, "397", "50");
            Thread.Sleep(100);
            if (nextluot) return;
            //Bat2FA.AdbHelper.ForceApp(LDPath, deviceID, packagename);
            Thread.Sleep(100);
            //Bat2FA.AdbHelper.ClearDataApp(LDPath, deviceID, packagename);
            if (nextluot) return;
            Bat2FA.AdbHelper.OpenApp(LDPath, index, packagename);
            Thread.Sleep(100);
            //Bat2FA.AdbHelper.ForceApp(LDPath, deviceID, packagename);
            //Thread.Sleep(1000);
            //Bat2FA.AdbHelper.OpenApp(LDPath, index, packagename);
            //Thread.Sleep(1000);
            if (nextluot) return;
            Bat2FA.AdbHelper.Runcmd(LDPath, deviceID, "shell ime set com.android.adbkeyboard/.AdbIME");
            
            //Bat2FA.AdbHelper.sort(LDPath);
            string uid = dataGridView1.Rows[i].Cells["uid1"].Value.ToString().Split('|')[0];
            string pass = dataGridView1.Rows[i].Cells["uid1"].Value.ToString().Split('|')[1];
            string noidung;
            Bitmap screen;
            string room1 = Convert.ToString(room.Value);
            string pass1 = Convert.ToString(pw.Value);
            int demagain = 0;
            int demagainsv = Convert.ToInt32(dem_try.Value);
            int thoat = 0;

        log:
            if (demagain== demagainsv)
            {
                goto loi;
            }
            int num = 0;
            while (!isstop)


            {
                int demchon = 0;
                if (nextluot) return;
                noidung = Bat2FA.AdbHelper.dump(deviceID, LDPath);
                if (noidung.Contains("\"OK\""))
                {
                    Bat2FA.AdbHelper.getbounds(noidung, deviceID, "\"OK\"", true, LDPath);
                    Thread.Sleep(1000);
                }
                if (!noidung.Contains("/start_layout"))
                {

                    screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);
                    if (Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "dongy.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "dongy1.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, true, deviceID, "dongy2.png", screen, pathimage))
                    {
                        Thread.Sleep(500);
                    }
                    else
                    {
                        Trangthai(i, "loi --> open app lai ");
                        Thread.Sleep(100);
                        Bat2FA.AdbHelper.ForceApp(LDPath, deviceID, packagename);
                        Thread.Sleep(500);
                        Bat2FA.AdbHelper.OpenApp(LDPath, index, packagename);
                        Thread.Sleep(5000);
                        goto log;
                    }
                    screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);
                    if (Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "dongy.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "dongy1.png", screen, pathimage))
                    {
                        Bat2FA.AdbHelper.tap(LDPath, deviceID, "350", "180");
                        Thread.Sleep(400);
                        Bat2FA.AdbHelper.tap(LDPath, deviceID, "300", "80");
                        Thread.Sleep(400);
                        while (!isstop)
                        {
                            if (Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "user.png", screen, pathimage))
                                Thread.Sleep(500);
                            Bat2FA.AdbHelper.tap(LDPath, deviceID, "244.0", "122.3");
                            Thread.Sleep(500);
                            Bat2FA.AdbHelper.Runcmd(LDPath, deviceID, "shell input keyevent --longpress 67 67 67 67 67 67 67 67 67 67 67 67");
                            Thread.Sleep(300);
                            {
                            em:
                                Thread.Sleep(500);
                                Trangthai(i, "Nhập Email");
                                Bat2FA.AdbHelper.inputtext(LDPath, deviceID, uid);
                                Thread.Sleep(500);
                            pa:
                                Thread.Sleep(500);
                                Bat2FA.AdbHelper.tap(LDPath, deviceID, "169.0", "152.3");
                                Thread.Sleep(300);
                                Bat2FA.AdbHelper.tap(LDPath, deviceID, "169.0", "152.3");
                                Thread.Sleep(500);
                                Bat2FA.AdbHelper.Runcmd(LDPath, deviceID, "shell input keyevent --longpress 67 67 67 67 67 67 67 67 67 67 67 67");
                                Thread.Sleep(300);
                                {


                                    Thread.Sleep(100);
                                    Trangthai(i, "Nhập Password");
                                    Bat2FA.AdbHelper.inputtext(LDPath, deviceID, pass);
                                    Thread.Sleep(200);


                                    Trangthai(i, "login");
                                    Bat2FA.AdbHelper.tap(LDPath, deviceID, "235.0", "237.3");
                                    Thread.Sleep(200);
                                    Bat2FA.AdbHelper.tap(LDPath, deviceID, "235.0", "237.3");
                                    Thread.Sleep(1000);
                                    int chuplai = 0;
                                    int loi1 = 0;

                                try1:
                                    screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);
                                    if (Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "dangoi.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "doiban.png", screen, pathimage))
                                    {

                                        Thread.Sleep(500);
                                        goto ngoiroom;
                                    }
                                    Bat2FA.AdbHelper.tap(LDPath, deviceID, "231", "241");
                                    Thread.Sleep(200);
                                    Bat2FA.AdbHelper.tap(LDPath, deviceID, "231", "241");
                                    Thread.Sleep(200);
                                    Bat2FA.AdbHelper.tap(LDPath, deviceID, "231", "241");
                                    Thread.Sleep(100);

                                    Trangthai(i, "xac nhan login");
                                    Thread.Sleep(200);
                                    screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);
                                    if (Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "sanpt.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "loginok.png", screen, pathimage))
                                    {
                                        Trangthai(i, " login ok 1");
                                        Thread.Sleep(500);
                                        goto vaoroom;
                                    }
                                    if (Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "dongy.png", screen, pathimage))
                                    {
                                        Trangthai(i, " login fail" + chuplai + "lan");
                                        chuplai++;
                                        Thread.Sleep(500);
                                        if (chuplai > 4)
                                        {
                                            demagain++;

                                            goto log;
                                        }
                                        else
                                        { goto try1; }

                                    }

                                    else
                                    {
                                        loi1++;
                                        Trangthai(i, "xac nhan lai login " + loi1 + "lan");
                                        Thread.Sleep(500);
                                        if (loi1 > 5)
                                        {
                                            Trangthai(i, "loi --> open app lai ");
                                            Thread.Sleep(1000);
                                            Bat2FA.AdbHelper.ForceApp(LDPath, deviceID, packagename);
                                            Thread.Sleep(1000);
                                            Bat2FA.AdbHelper.OpenApp(LDPath, index, packagename);
                                            Thread.Sleep(1000);
                                            goto log;
                                        }
                                        else
                                        { goto try1; }

                                    }

                                    // chonroom:

                                    // Trangthai(i, "chon phong");
                                    // Bat2FA.AdbHelper.tap(LDPath, deviceID, "231", "241");
                                    // Thread.Sleep(100);
                                    // Thread.Sleep(100);
                                    // Bat2FA.AdbHelper.tap(LDPath, deviceID, "231", "241");
                                    // Thread.Sleep(100);
                                    //Bat2FA.AdbHelper.tap(LDPath, deviceID, "231", "241");
                                    // Thread.Sleep(100);
                                    // timpt:
                                    // Trangthai(i, "tim san pt");
                                    // screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);
                                    // if (Bat2FA.AdbHelper.CheckExistImage(LDPath, true, deviceID, "sanpt.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, true, deviceID, "sanpt1.png", screen, pathimage))
                                    //{
                                    //     Thread.Sleep(500);

                                    // }

                                    // timroom:
                                    //Trangthai(i, "tim room");
                                    // Thread.Sleep(500);
                                    //  screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);
                                    // if (Bat2FA.AdbHelper.CheckExistImage(LDPath, true, deviceID, "timroom.png", screen, pathimage))
                                    //{
                                    //     Thread.Sleep(500);

                                    // }

                                    // nhaproom:
                                    // screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);
                                    // if (Bat2FA.AdbHelper.CheckExistImage(LDPath, true, deviceID, "nhaproom.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, true, deviceID, "nhaproom1.png", screen, pathimage))
                                    // {
                                    //     Thread.Sleep(500);

                                    // }



                                    //Trangthai(i, "nhap room");
                                    // Thread.Sleep(500);
                                    // Bat2FA.AdbHelper.inputtext(LDPath, deviceID, room1);
                                    // Thread.Sleep(500);
                                    // Bat2FA.AdbHelper.tap(LDPath, deviceID, "426", "255");
                                    //Thread.Sleep(500);
                                    //timkiem:
                                    // screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);
                                    //if (Bat2FA.AdbHelper.CheckExistImage(LDPath, true, deviceID, "timkiem.png", screen, pathimage))
                                    // {
                                    //    Thread.Sleep(500);

                                    //}
                                    //else { goto timkiem; }
                                    // Bat2FA.AdbHelper.tap(LDPath, deviceID, "290", "93");
                                    //  Thread.Sleep(500);
                                    // timpw:
                                    // if (Bat2FA.AdbHelper.CheckExistImage(LDPath, true, deviceID, "timpass.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, true, deviceID, "timpass1.png", screen, pathimage))
                                    // {
                                    //    Thread.Sleep(500);
                                    // Bat2FA.AdbHelper.tap(LDPath, deviceID, "219", "172");
                                    //  Trangthai(i, "nhap pass");
                                    //   Thread.Sleep(1000);
                                    // Bat2FA.AdbHelper.inputtext(LDPath, deviceID, pass1);
                                    //  Thread.Sleep(1500);

                                    // }
                                    // vaoroom:
                                    //  screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);
                                    // if (Bat2FA.AdbHelper.CheckExistImage(LDPath, true, deviceID, "vaoroom.png", screen, pathimage))
                                    // {
                                    //  Thread.Sleep(500);
                                    //   Bat2FA.AdbHelper.tap(LDPath, deviceID, "236", "213");
                                    //}
                                    //else { goto vaoroom; }
                                    //again:
                                    // Trangthai(i, "again");

                                    // Thread.Sleep(1000);
                                    // Bat2FA.AdbHelper.tap(LDPath, deviceID, "373.0", "180.3");
                                    // Thread.Sleep(500);
                                    // Bat2FA.AdbHelper.tap(LDPath, deviceID, "231.0", "122.3");
                                    //Thread.Sleep(500);
                                    // Bat2FA.AdbHelper.Runcmd(LDPath, deviceID, "shell input keyevent --longpress 67 67 67 67 67 67 67 67 67 67 67 67");
                                    //Thread.Sleep(300);
                                    //{

                                    //  Thread.Sleep(500);
                                    //  Trangthai(i, "Nhập Email");
                                    // Bat2FA.AdbHelper.inputtext(LDPath, deviceID, uid);
                                    //Thread.Sleep(500);

                                    //Thread.Sleep(500);
                                    // Bat2FA.AdbHelper.tap(LDPath, deviceID, "169.0", "152.3");
                                    //Thread.Sleep(300);
                                    ///  Bat2FA.AdbHelper.tap(LDPath, deviceID, "169.0", "152.3");
                                    // Thread.Sleep(500);
                                    // Bat2FA.AdbHelper.Runcmd(LDPath, deviceID, "shell input keyevent --longpress 67 67 67 67 67 67 67 67 67 67 67 67");
                                    // Thread.Sleep(300);
                                    // {
                                    //      

                                    // /  Thread.Sleep(100);
                                    // / Trangthai(i, "Nhập Password");
                                    //  Bat2FA.AdbHelper.inputtext(LDPath, deviceID, pass);
                                    // Thread.Sleep(500);
                                    //Bat2FA.AdbHelper.tap(LDPath, deviceID, "235.0", "237.3");
                                    //Thread.Sleep(200);
                                    //Bat2FA.AdbHelper.tap(LDPath, deviceID, "235.0", "237.3");
                                    // Thread.Sleep(1000);
                                    // screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);
                                    // if (Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "dongy.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "dongy1.png", screen, pathimage))
                                    //{
                                    //     goto loi;
                                    // }
                                    //}


                                }

                            }
                        }
                    vaoroom:
                        while (!isstop)

                        {
                            Trangthai(i, "dang nhap ok--> vao san PT");

                            screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);
                            if (Bat2FA.AdbHelper.CheckExistImage(LDPath, true, deviceID, "sanpt.png", screen, pathimage))
                            { Thread.Sleep(500); }
                            screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);
                            if (Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "sansocap.png", screen, pathimage))
                            { break; }

                        }
                    timroom:
                        if (vaoroom.Checked)
                        {
                            while (!isstop)

                            {
                                Trangthai(i, "timroom");
                                Thread.Sleep(500);
                                screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);
                                Thread.Sleep(500);
                                if (Bat2FA.AdbHelper.CheckExistImage(LDPath, true, deviceID, "timroom.png", screen, pathimage))
                                {
                                    Thread.Sleep(500);

                                }
                                screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);
                                Thread.Sleep(500);
                                if (Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "nhaproom.png", screen, pathimage))
                                { break; }

                            }
                        nhaproom:
                            while (!isstop)

                            {
                                Trangthai(i, " click vao nhap room");
                                Thread.Sleep(500);
                                screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);
                                Thread.Sleep(500);
                                if (Bat2FA.AdbHelper.CheckExistImage(LDPath, true, deviceID, "nhaproom.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, true, deviceID, "nhaproom1.png", screen, pathimage))
                                {
                                    Thread.Sleep(500);

                                }
                                screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);
                                if (Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "daclick.png", screen, pathimage))
                                {
                                    break;
                                }
                            }
                            while (!isstop)

                            {
                                Trangthai(i, "nhap room");

                                Thread.Sleep(500);
                                Bat2FA.AdbHelper.inputtext(LDPath, deviceID, room1);
                                Thread.Sleep(500);
                                Bat2FA.AdbHelper.tap(LDPath, deviceID, "426", "255");
                                Thread.Sleep(500);
                           
                                screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);
                                Thread.Sleep(500);
                                if (Bat2FA.AdbHelper.CheckExistImage(LDPath, true, deviceID, "timkiem.png", screen, pathimage))
                                {
                                    Thread.Sleep(500);

                                }
                                screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);
                                Thread.Sleep(500);
                                if (Bat2FA.AdbHelper.CheckExistImage(LDPath, true, deviceID, "cophong.png", screen, pathimage))
                                {
                                    Thread.Sleep(500);
                                    break;
                                }
                                else
                                { goto timroom; }

                            }

                        timpw:
                            while (!isstop)

                            {
                                Trangthai(i, "nhap pass");
                                Thread.Sleep(100);
                                if (Bat2FA.AdbHelper.CheckExistImage(LDPath, true, deviceID, "chonroom.png", screen, pathimage)) ;
                                screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);
                                Thread.Sleep(500);
                                if (Bat2FA.AdbHelper.CheckExistImage(LDPath, true, deviceID, "timpass.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, true, deviceID, "timpass1.png", screen, pathimage))
                                {
                                    {
                                        Thread.Sleep(1000);
                                        Bat2FA.AdbHelper.tap(LDPath, deviceID, "219", "172");

                                        Thread.Sleep(1000);
                                        Bat2FA.AdbHelper.inputtext(LDPath, deviceID, pass1);
                                        Thread.Sleep(100);

                                    }
                                    screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);
                                    Thread.Sleep(500);
                                    if (Bat2FA.AdbHelper.CheckExistImage(LDPath, true, deviceID, "vaoroom.png", screen, pathimage))
                                    {
                                        Thread.Sleep(500);
                                        Bat2FA.AdbHelper.tap(LDPath, deviceID, "236", "213");
                                    }
                                    goto allin;
                                }


                            }
                        }

                    sansocap:

                        if (choitudo.Checked)
                        {


                            while (!isstop)

                            {

                                if (Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "lag.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "khongphanhoi.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "home.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "loiht.png", screen, pathimage))
                                {
                                    Thread.Sleep(2000);
                                    screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);
                                    if (Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "lag.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "khongphanhoi.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "home.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "home.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, true, deviceID, "loiht.png", screen, pathimage))
                                    {
                                        Trangthai(i, "loi --> open app lai ");
                                        Thread.Sleep(1000);
                                        Bat2FA.AdbHelper.ForceApp(LDPath, deviceID, packagename);
                                        Thread.Sleep(1000);
                                        Bat2FA.AdbHelper.OpenApp(LDPath, index, packagename);
                                        Thread.Sleep(1000);
                                        goto log;
                                    }
                                }
                                else
                                {
                                    Thread.Sleep(100);
                                }
                                if (demchon > 4)
                                {
                                    demchon = 0;
                                }
                                Trangthai(i, +demchon + "vao san so cap");

                                screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);
                                Thread.Sleep(500);
                                if (Bat2FA.AdbHelper.CheckExistImage(LDPath, true, deviceID, "sansocap.png", screen, pathimage))
                                {
                                    {

                                        Thread.Sleep(300);

                                    }
                                    screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);
                                    if (Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "davaosocap.png", screen, pathimage))
                                    {
                                        Trangthai(i, "click vao room");
                                        Thread.Sleep(500);
                                        if (demchon == 0)
                                        {
                                            Bat2FA.AdbHelper.tap(LDPath, deviceID, "242", "254");
                                        }
                                        if (demchon == 1)
                                        {
                                            Bat2FA.AdbHelper.tap(LDPath, deviceID, "285", "216");
                                        }
                                        if (demchon == 2)
                                        {
                                            Bat2FA.AdbHelper.tap(LDPath, deviceID, "234", "174");
                                        }
                                        if (demchon == 3)
                                        {
                                            Bat2FA.AdbHelper.tap(LDPath, deviceID, "254", "131");
                                        }
                                        if (demchon == 4)
                                        {
                                            Bat2FA.AdbHelper.tap(LDPath, deviceID, "233", "87");
                                        }
                                        Thread.Sleep(500);

                                    }

                                }
                                screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);
                                if (Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "davao.png", screen, pathimage))
                                { break; }
                                if (Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "dangoi.png", screen, pathimage))
                                { break; }
                                if (Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "doiban.png", screen, pathimage))
                                { break; }
                                
                                else

                                {
                                    demchon++;
                                    goto sansocap;
                                }
                            }
                        }
                    ngoiroom:
                        while (!isstop)

                        {
                            Trangthai(i, "co cho ngoi chua?");
                            Thread.Sleep(2000);

                            if (Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "lag.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "khongphanhoi.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "home.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "loiht.png", screen, pathimage))
                            {
                                Thread.Sleep(2000);
                                screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);
                                if (Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "lag.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "khongphanhoi.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "home.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "home.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, true, deviceID, "loiht.png", screen, pathimage))
                                {
                                    Trangthai(i, "loi --> open app lai ");
                                    Thread.Sleep(1000);
                                    Bat2FA.AdbHelper.ForceApp(LDPath, deviceID, packagename);
                                    Thread.Sleep(1000);
                                    Bat2FA.AdbHelper.OpenApp(LDPath, index, packagename);
                                    Thread.Sleep(1000);
                                    goto log;
                                }
                            }
                            else
                            {
                                Thread.Sleep(100);
                            }

                            if (Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "dangoi.png", screen, pathimage))
                            {
                                Thread.Sleep(500);
                                screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);
                                if (Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "khongconguoi.png", screen, pathimage))
                                {
                                    Thread.Sleep(500);
                                    screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);
                                    if (Bat2FA.AdbHelper.CheckExistImage(LDPath, true, deviceID, "dangoi.png", screen, pathimage)) ;
                                    Thread.Sleep(500);
                                    Bat2FA.AdbHelper.Runcmd(LDPath, deviceID, "shell input keyevent 4");
                                    Thread.Sleep(500);
                                    goto sansocap;
                                }
                                screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);
                                if (Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "doiban.png", screen, pathimage))
                                {
                                    Thread.Sleep(500);
                                    screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);
                                    if (Bat2FA.AdbHelper.CheckExistImage(LDPath, true, deviceID, "dangoi.png", screen, pathimage)) ;
                                    Thread.Sleep(500);
                                    Bat2FA.AdbHelper.Runcmd(LDPath, deviceID, "shell input keyevent 4");
                                    Thread.Sleep(500);
                                    goto sansocap;
                                }
                                else
                                { break; }
                            }

                            else
                            {
                                Thread.Sleep(500);
                                Bat2FA.AdbHelper.Runcmd(LDPath, deviceID, "shell input keyevent 4");
                                Thread.Sleep(500);
                                goto sansocap;
                            }
                        }
                        int delayplay = Convert.ToInt32(doi_auto.Value);

                    autoplay:



                        while (!isstop)
                        {
                            delayplay--;
                            Trangthai(i, " Đợi auto play : " + delayplay + " giây");
                            Thread.Sleep(500);
                            if (delayplay < 1)
                            {
                                Thread.Sleep(500);
                                goto thoat;
                            }
                            screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);
                            if (Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "khongconguoi.png", screen, pathimage))
                            {
                                Thread.Sleep(500);
                                if (Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "dangoi.png", screen, pathimage)) ;
                                Thread.Sleep(500);
                                Bat2FA.AdbHelper.Runcmd(LDPath, deviceID, "shell input keyevent 4");
                                Thread.Sleep(500);
                                goto sansocap;
                            }
                            if (Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "lag.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "khongphanhoi.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "home.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "loiht.png", screen, pathimage))
                            {
                                Thread.Sleep(2000);
                                screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);
                                if (Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "lag.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "khongphanhoi.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "home.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "home.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, true, deviceID, "loiht.png", screen, pathimage))
                                {
                                    Trangthai(i, "loi --> open app lai ");
                                    Thread.Sleep(1000);
                                    Bat2FA.AdbHelper.ForceApp(LDPath, deviceID, packagename);
                                    Thread.Sleep(1000);
                                    Bat2FA.AdbHelper.OpenApp(LDPath, index, packagename);
                                    Thread.Sleep(1000);
                                    goto log;
                                }
                            }
                            else
                            {
                                Thread.Sleep(100);
                            }
                            Thread.Sleep(500);
                            screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);

                            if (Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "denluot.png", screen, pathimage))
                            {


                                Thread.Sleep(100);


                                if (Bat2FA.AdbHelper.CheckExistImage(LDPath, true, deviceID, "xembai.png", screen, pathimage))

                                {
                                    Thread.Sleep(100);
                                }
                                else
                                if (Bat2FA.AdbHelper.CheckExistImage(LDPath, true, deviceID, "denluot.png", screen, pathimage)) ;
                            }

                            else
                            { goto autoplay; }

                        }
                    allin:
                        if (vaoroom.Checked)
                        {
                            while (!isstop)
                            {

                                if (Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "lag.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "khongphanhoi.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "home.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "loiht.png", screen, pathimage))
                                {
                                    Thread.Sleep(2000);
                                    screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);
                                    if (Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "lag.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "khongphanhoi.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "home.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "home.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, true, deviceID, "loiht.png", screen, pathimage))
                                    {
                                        Trangthai(i, "loi --> open app lai ");
                                        Thread.Sleep(1000);
                                        Bat2FA.AdbHelper.ForceApp(LDPath, deviceID, packagename);
                                        Thread.Sleep(1000);
                                        Bat2FA.AdbHelper.OpenApp(LDPath, index, packagename);
                                        Thread.Sleep(1000);
                                        goto log;
                                    }
                                }
                                else
                                {
                                    Thread.Sleep(100);
                                }
                                Trangthai(i, " all ");
                                Thread.Sleep(500);
                                screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);
                                Thread.Sleep(500);
                                if (Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "dangoi.png", screen, pathimage)) ;
                                Thread.Sleep(500);
                            all1:
                                Trangthai(i, " cho den luot ");
                                screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);
                                Thread.Sleep(500);
                                if (Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "hetchip.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "hetchip1.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, true, deviceID, "hetchip2.png", screen, pathimage))
                                {
                                    Trangthai(i, " het chip ");
                                    Thread.Sleep(500);
                                    goto xong;
                                }
                                else
                                {

                                    Thread.Sleep(100);
                                }
                               
                                Thread.Sleep(500);
                                screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);
                                if (Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "denluot.png", screen, pathimage))
                                {
                                    Trangthai(i, " da den luot ");
                                    Thread.Sleep(500);
                                    if (Bat2FA.AdbHelper.CheckExistImage(LDPath, true, deviceID, "themcuoc.png", screen, pathimage))

                                    {
                                        Thread.Sleep(500);
                                        Bat2FA.AdbHelper.tap(LDPath, deviceID, "417", "136");
                                    }
                                    if (Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "themng.png", screen, pathimage))

                                    {
                                        Thread.Sleep(500);
                                        Bat2FA.AdbHelper.tap(LDPath, deviceID, "340", "305");
                                    }
                                    else
                                    {
                                        Thread.Sleep(100);
                                    }

                                }

                                else
                                { goto all1; }

                            }
                        }
                    
                        sancaocap:
                        while (!isstop)

                        {
                            Trangthai(i, "vao san cao cap");

                            screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);
                            Thread.Sleep(500);
                            if (Bat2FA.AdbHelper.CheckExistImage(LDPath, true, deviceID, "sancaocap.png", screen, pathimage))
                            {
                                {

                                    Thread.Sleep(100);

                                }
                                break;
                            }
                            if (Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "lag.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "khongphanhoi.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "home.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "loiht.png", screen, pathimage))
                            {
                                Thread.Sleep(2000);
                                screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);
                                if (Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "lag.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "khongphanhoi.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "home.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "home.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, true, deviceID, "loiht.png", screen, pathimage))
                                {
                                    Trangthai(i, "loi --> open app lai ");
                                    Thread.Sleep(1000);
                                    Bat2FA.AdbHelper.ForceApp(LDPath, deviceID, packagename);
                                    Thread.Sleep(1000);
                                    Bat2FA.AdbHelper.OpenApp(LDPath, index, packagename);
                                    Thread.Sleep(1000);
                                    goto log;
                                }
                            }
                            else
                            {
                                Thread.Sleep(100);
                            }
                        }
                    thoat:


                        Trangthai(i, "thoat" + thoat + "lan");
                        while (!isstop)
                        {

                            if (Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "lag.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "khongphanhoi.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "home.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "loiht.png", screen, pathimage))
                            {
                                Thread.Sleep(2000);
                                screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);
                                if (Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "lag.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "khongphanhoi.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "home.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, false, deviceID, "home.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, true, deviceID, "loiht.png", screen, pathimage))
                                {
                                    Trangthai(i, "loi --> open app lai ");
                                    Thread.Sleep(1000);
                                    Bat2FA.AdbHelper.ForceApp(LDPath, deviceID, packagename);
                                    Thread.Sleep(1000);
                                    Bat2FA.AdbHelper.OpenApp(LDPath, index, packagename);
                                    Thread.Sleep(1000);
                                    goto log;
                                }
                            }
                            else
                            {
                                Thread.Sleep(100);
                            }
                            thoat++;

                            Thread.Sleep(500);
                            if (thoat > 5)
                            {
                                Trangthai(i, "loi --> open app lai ");
                                Thread.Sleep(1000);
                                Bat2FA.AdbHelper.ForceApp(LDPath, deviceID, packagename);
                                Thread.Sleep(1000);
                                Bat2FA.AdbHelper.OpenApp(LDPath, index, packagename);
                                Thread.Sleep(1000);
                                goto xong;
                            }
                            {
                                Trangthai(i, "thoat" + thoat + "lan");
                                Bat2FA.AdbHelper.Runcmd(LDPath, deviceID, "shell input keyevent 4");
                                Thread.Sleep(1000);
                                screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);
                                if (Bat2FA.AdbHelper.CheckExistImage(LDPath, true, deviceID, "xacnhandung.png", screen, pathimage))
                                {
                                    Thread.Sleep(500);
                                    Bat2FA.AdbHelper.tap(LDPath, deviceID, "290", "212");
                                }


                            outtk:
                                Thread.Sleep(1000);
                                screen = Bat2FA.AdbHelper.capturescreen(deviceID, LDPath);

                                if (Bat2FA.AdbHelper.CheckExistImage(LDPath, true, deviceID, "thoat.png", screen, pathimage) || Bat2FA.AdbHelper.CheckExistImage(LDPath, true, deviceID, "thoat1.png", screen, pathimage))
                                {
                                    Thread.Sleep(500);
                                    Bat2FA.AdbHelper.tap(LDPath, deviceID, "285", "243");
                                }
                                else
                                {
                                    Bat2FA.AdbHelper.Runcmd(LDPath, deviceID, "shell input keyevent 4");
                                    goto outtk;
                                }

                                goto xong;
                            }
                        }
                    }
                   

                }
            

                num++;
                Thread.Sleep(100);
            }
            num = 0;
            
        ne:
            Thread.Sleep(100);


        }


        public string Randomstr()
        {
            string a = "abcdefghiklmnopqrstuvwzy";
            Random rd = new Random();
            string b = "";
            for (int i = 0; i < 10; i++)
            {
                if (rd.Next(0, 2) == 0)
                {
                    b = b + a[rd.Next(0, a.Length)];
                }
                else
                {
                    b = b + a[rd.Next(0, a.Length)].ToString().ToUpper();
                }
            }
            return b;
        }







        string linkreset = "";
        //HEETS MAIN
        public void Trangthai(int i, string text)
        {
            dataGridView1.Rows[i].Cells["trangthai1"].Value = text;
        }
        public List<string> GetDevice()
        {
            List<string> listdv1 = new List<string>();
            try
            {
                string listdv = Bat2FA.AdbHelper.Getstring(LDPath, "adb devices");
                if (listdv != "")
                {
                    string[] list = listdv.Split(new string[] { Environment.NewLine }, StringSplitOptions.RemoveEmptyEntries);
                    for (int i = 0; i < list.Length; i++)
                    {
                        if (list[i].Contains("\tdevice"))
                        {
                            listdv1.Add(list[i].Replace("\tdevice", ""));
                        }
                    }

                }
                else
                {
                }
            }
            catch { }
            return listdv1;

        }
        public void LoadDevices()
        {
            List<string> listdv = GetDevice();
            if (listdv.Count == 0)
            {
                MessageBox.Show("KHÔNG TÌM THẤY LD NÀO MỞ !");
            }
            else
            {
                foreach (var t in listdv)
                {
                    string nameld = "";
                    if (t.ToString().Contains("127.0") || t.ToString().Contains("emulator-"))
                    {
                        queld.Add(t.ToString());
                    }

                }
            }
        }
        List<string> queld = new List<string>();
        public bool Check()
        {
            LDPath = File.ReadAllText("LDPath.txt");
            if (LDPath == "")
            {
                MessageBox.Show("CHƯA NHẬP ĐƯỜNG DẪN LDPLAYER!");
                return false;
            }
            else if (LDPath.Contains(" "))
            {
                MessageBox.Show("ĐƯỜNG DẪN KHÔNG ĐƯỢC CHỨA DẤU CÁCH, KÝ TỰ ĐẶC BIỆT, CHỮ TIẾNG VIỆT!");
                return false;
            }
            queld = new List<string>();
            LoadDevices();
            if (queld.Count == 0)
            {
                return false;
            }
            if (dataGridView1.Rows.Count == 0)
            {
                MessageBox.Show("CHƯA NHẬP ACCOUNT!");
                return false;
            }
            listld = Bat2FA.AdbHelper.GetIndexLD(LDPath);
            return true;
        }

        private void button6_Click(object sender, EventArgs e)
        {
            Process.Start("explorer.exe", AppDomain.CurrentDomain.BaseDirectory + "OUTPUT\\OUTPUT.txt");
        }

        private void btn_stop_Click(object sender, EventArgs e)
        {
            isstop = true;
            try
            {
                //foreach (int ind in listindex)
                //{
                //    Bat2FA.AdbHelper.QuitLD(LDPath, ind);
                //}
                //Bat2FA.AdbHelper.QuitAllLD(LDPath);
                foreach (Thread t in listthread)
                {
                    try
                    {
                        t.Abort();
                    }
                    catch { }
                }
            }
            catch { }
            Thread.Sleep(4000);
            Invoke(new Action(() =>
            {
                btn_start.Enabled = true;
                btn_start.BackColor = Color.Green;
                btn_stop.Enabled = false;
                btn_stop.BackColor = Color.Gray;
            }));

            //MessageBox.Show("XONG");
        }

        private void uIDPASS2FAToolStripMenuItem_Click(object sender, EventArgs e)
        {
            string dt = "";
            for (int i = dataGridView1.SelectedRows.Count - 1; i >= 0; i--)
            {
                dt += dataGridView1.SelectedRows[i].Cells["uid1"].Value.ToString() + "|" +
                    dataGridView1.SelectedRows[i].Cells["pass1"].Value.ToString() + "|" +
                    dataGridView1.SelectedRows[i].Cells["ma2fa1"].Value.ToString() + "\r\n";
            }
            Clipboard.SetText(dt);
            MessageBox.Show("XONG");
        }

        private void uIDPASS2FACOOKIETOKENToolStripMenuItem_Click(object sender, EventArgs e)
        {
            string dt = "";
            for (int i = dataGridView1.SelectedRows.Count - 1; i >= 0; i--)
            {
                dt += dataGridView1.SelectedRows[i].Cells["uid1"].Value.ToString() + "|" + dataGridView1.SelectedRows[i].Cells["pass1"].Value.ToString() + "|" + dataGridView1.SelectedRows[i].Cells["ma2fa1"].Value.ToString()
                    + "|" + dataGridView1.SelectedRows[i].Cells["cookie1"].Value.ToString() + "|" + dataGridView1.SelectedRows[i].Cells["token1"].Value.ToString() + "\r\n";
            }
            Clipboard.SetText(dt);
            MessageBox.Show("XONG");
        }

        private void button1_Click(object sender, EventArgs e)
        {
            dataGridView1.Rows.Clear();
        }

        private void Form1_Load(object sender, EventArgs e)
        {

        }

        private void xÓAHÀNGBÔIĐENToolStripMenuItem_Click(object sender, EventArgs e)
        {
            DialogResult rs = MessageBox.Show("Bạn chắc chắn muốn xóa??", "", MessageBoxButtons.YesNo, MessageBoxIcon.Warning);
            if (rs == DialogResult.Yes)
            {
                foreach (DataGridViewRow row in dataGridView1.SelectedRows)
                {
                    dataGridView1.Rows.Remove(row);
                }
                SaveData();
            }

        }


        private void uIDPASS2FACOOKIEToolStripMenuItem_Click(object sender, EventArgs e)
        {
            string dt = "";
            for (int i = dataGridView1.SelectedRows.Count - 1; i >= 0; i--)
            {
                dt += dataGridView1.SelectedRows[i].Cells["uid1"].Value.ToString() + "|" + dataGridView1.SelectedRows[i].Cells["pass1"].Value.ToString() + "|" + dataGridView1.SelectedRows[i].Cells["ma2fa1"].Value.ToString()
                    + "|" + dataGridView1.SelectedRows[i].Cells["cookie1"].Value.ToString() + "\r\n";
            }
            Clipboard.SetText(dt);
            MessageBox.Show("XONG");
        }

        private void xÓAHÀNGTICKCHỌNToolStripMenuItem_Click(object sender, EventArgs e)
        {
            DialogResult rs = MessageBox.Show("Bạn chắc chắn muốn xóa??", "", MessageBoxButtons.YesNo, MessageBoxIcon.Warning);
            if (rs == DialogResult.Yes)
            {
                for (int i = 0; i < dataGridView1.Rows.Count; i++)
                {
                    if (dataGridView1.Rows[i].Cells[0].Value.Equals(true))
                    {
                        dataGridView1.Rows.RemoveAt(i);
                        i--;
                    }

                }
                SaveData();
            }

        }

        private void cHỌNACCCHECKPOINTToolStripMenuItem_Click(object sender, EventArgs e)
        {
            foreach (DataGridViewRow row in dataGridView1.Rows)
            {
                if (row.Cells["trangthai1"].Value.ToString().Contains("Checkpoint"))
                {
                    row.Cells[0].Value = true;
                }
                else
                {
                    row.Cells[0].Value = false;
                }
            }

        }

        private void cHỌNACCSAIPASSToolStripMenuItem_Click(object sender, EventArgs e)
        {
            foreach (DataGridViewRow row in dataGridView1.Rows)
            {
                if (row.Cells["trangthai1"].Value.ToString().Contains("Sai pass"))
                {
                    row.Cells[0].Value = true;
                }
                else
                {
                    row.Cells[0].Value = false;
                }
            }
        }

        private void cHỌNACCSAI2FAToolStripMenuItem_Click(object sender, EventArgs e)
        {
            foreach (DataGridViewRow row in dataGridView1.Rows)
            {
                if (row.Cells["trangthai1"].Value.ToString().Contains("Sai 2FA"))
                {
                    row.Cells[0].Value = true;
                }
                else
                {
                    row.Cells[0].Value = false;
                }
            }
        }

        private void pASSToolStripMenuItem_Click(object sender, EventArgs e)
        {
            string dt = Clipboard.GetText();
            foreach (DataGridViewRow row in dataGridView1.Rows)
            {
                if (row.Cells[0].Value.Equals(true))
                {
                    row.Cells["pass1"].Value = dt;
                }
            }
            SaveData();

        }

        private void fAToolStripMenuItem_Click(object sender, EventArgs e)
        {
            string dt = Clipboard.GetText();
            foreach (DataGridViewRow row in dataGridView1.Rows)
            {
                if (row.Cells[0].Value.Equals(true))
                {
                    row.Cells["ma2fa1"].Value = dt;
                }
            }
            SaveData();
        }

        private void tHÊMCHOTẤTCẢToolStripMenuItem_Click(object sender, EventArgs e)
        {
            string[] listproxy = Clipboard.GetText().Split(new string[] { Environment.NewLine }, StringSplitOptions.RemoveEmptyEntries).ToArray();
            int k = 0;
            for (int i = 0; i < dataGridView1.Rows.Count; i++)
            {
                if (k != listproxy.Length - 1)
                {
                    dataGridView1.Rows[i].Cells["proxy1"].Value = listproxy[k];
                    k++;
                }
                else
                {
                    dataGridView1.Rows[i].Cells["proxy1"].Value = listproxy[k];
                    k = 0;
                }
            }
        }

        private void tHÊMCHOHÀNGBÔIĐENToolStripMenuItem_Click(object sender, EventArgs e)
        {
            foreach (DataGridViewRow row in dataGridView1.SelectedRows)
            {
                row.Cells["proxy1"].Value = Clipboard.GetText();
            }
        }

        private void button3_Click_1(object sender, EventArgs e)
        {
            if (!File.Exists("linkreset.txt"))
            {
                File.WriteAllText("linkreset.txt", "");
            }
            Process.Start("explorer.exe", AppDomain.CurrentDomain.BaseDirectory + "linkreset.txt");
        }

        private void button3_Click_2(object sender, EventArgs e)
        {
            Bat2FA.AdbHelper.QuitAllLD(LDPath);
        }
        bool outld = false;
        private void btn_out_Click(object sender, EventArgs e)
        {
            outld = true;
        }

        private void cOPYACCToolStripMenuItem_Click(object sender, EventArgs e)
        {
            StringBuilder acc = new StringBuilder();
            foreach (DataGridViewRow row in dataGridView1.SelectedRows)
            {
                acc.Append(row.Cells["uid1"].Value.ToString() + "\r\n");
            }
            Clipboard.SetText(acc.ToString());
            MessageBox.Show("OK");
        }

        private void cb_xemid_CheckedChanged(object sender, EventArgs e)
        {

        }

        private void label1_Click(object sender, EventArgs e)
        {

        }
    }
}
