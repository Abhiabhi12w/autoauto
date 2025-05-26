using System;
using System.Data;
using System.IO;
using System.Windows.Forms;
using System.Linq;

namespace CsvReaderApp
{
    public partial class Form1 : Form
    {

        private string configPath;
        private string currentCsvPath;


        public Form1()
        {
            InitializeComponent();

            tdmRunTextBox.Enabled = false; // Disable editin
            carryOnTestCaseComboBox.SelectedIndex = 1; // Default to FALSE
            carryOnTestCaseComboBox.Enabled = false; // Disable editing

        }

        private void btnBrowse_Click(object sender, EventArgs e)
        {
            OpenFileDialog openFileDialog = new OpenFileDialog();
            openFileDialog.Filter = "CSV files (*.csv)|*.csv";
            if (openFileDialog.ShowDialog() == DialogResult.OK)
            {
                txtLocation.Text = openFileDialog.FileName;
                LoadCsv(openFileDialog.FileName);
            }
        }

        private void LoadCsv(string filePath)
        {
            currentCsvPath = filePath;
            DataTable dt = new DataTable();
            using (StreamReader sr = new StreamReader(filePath))
            {
                string[] headers = sr.ReadLine()?.Split(',');
                if (headers == null) return;

                int[] columnsToKeep = { 0, 2, 4, 5, 6, 7, 8, 9 };

                // Ensure all indices are within bounds
                columnsToKeep = columnsToKeep.Where(i => i < headers.Length).ToArray();

                foreach (int index in columnsToKeep)
                {
                    dt.Columns.Add(headers[index]);
                }

                while (!sr.EndOfStream)
                {
                    string[] rows = sr.ReadLine()?.Split(',');
                    if (rows == null || rows.Length < headers.Length) continue;

                    DataRow dr = dt.NewRow();
                    foreach (int index in columnsToKeep)
                    {
                        if (index < rows.Length)
                            dr[headers[index]] = rows[index];
                        else
                            dr[headers[index]] = ""; // Fill with empty string if missing
                    }
                    dt.Rows.Add(dr);
                }
            }
            dataGridView1.DataSource = dt;
        }


        private void btnSaveCsv_Click(object sender, EventArgs e)
        {
            if (dataGridView1.DataSource is DataTable dt && !string.IsNullOrEmpty(currentCsvPath))
            {
                using (StreamWriter sw = new StreamWriter(currentCsvPath, false))
                {
                    // Write headers
                    var columnNames = dt.Columns.Cast<DataColumn>().Select(col => col.ColumnName);
                    sw.WriteLine(string.Join(",", columnNames));

                    // Write rows
                    foreach (DataRow row in dt.Rows)
                    {
                        var fields = row.ItemArray.Select(field => field.ToString());
                        sw.WriteLine(string.Join(",", fields));
                    }
                }

                MessageBox.Show("CSV file saved successfully!", "Saved", MessageBoxButtons.OK, MessageBoxIcon.Information);
            }
            else
            {
                MessageBox.Show("No data to save or file path is missing.", "Error", MessageBoxButtons.OK, MessageBoxIcon.Warning);
            }
        }



        private void browseButton_Click(object sender, EventArgs e)
        {
            FolderBrowserDialog folderDialog = new FolderBrowserDialog();
            if (folderDialog.ShowDialog() == DialogResult.OK)
            {
                configPath = Path.Combine(folderDialog.SelectedPath, @"Data\Config\");
                pathTextBox.Text = configPath;
            }
        }

        private void updateButton_Click(object sender, EventArgs e)
        {
            if (string.IsNullOrEmpty(configPath)) return;

            string selectedEnvironment = envComboBox.SelectedItem?.ToString();
            string dbPassword = dbPasswordTextBox.Text;

            if (string.IsNullOrEmpty(selectedEnvironment) || string.IsNullOrEmpty(dbPassword))
            {
                MessageBox.Show("Please select an environment and enter a DB password!", "Error", MessageBoxButtons.OK, MessageBoxIcon.Warning);
                return;
            }

            string configFile = Path.Combine(configPath, "Config.csv");
            string envManagerFile = Path.Combine(configPath, "Environment_Manager.csv");

            var configData = File.ReadAllLines(configFile).Select(line => line.Split(',')).ToList();

            // Updating Config.csv
            for (int i = 0; i < configData.Count; i++)
            {
                if (configData[i][0] == "Environment") configData[i][1] = selectedEnvironment;
                if (configData[i][0] == "UserName") configData[i][1] = userNameTextBox.Text;
                if (configData[i][0] == "Password") configData[i][1] = passwordTextBox.Text;
                if (configData[i][0] == "DBPassword") configData[i][1] = dbPassword; // Updating DBPassword
                if (configData[i][0] == "TDMRun") configData[i][1] = "TRUE"; // Always TRUE
                if (configData[i][0] == "CarryOnTestCase") configData[i][1] = "FALSE"; // Always FALSE
                if (configData[i][0] == "Retries") configData[i][1] = "6"; // Always 6
            }
            File.WriteAllLines(configFile, configData.Select(line => string.Join(",", line)));

            // Updating Environment_Manager.csv
            var envManagerData = File.ReadAllLines(envManagerFile).Select(line => line.Split(',')).ToList();
            int envColumnIndex = -1;

            // Find the column index for the selected environment
            string[] headers = envManagerData[0]; // First row contains headers
            for (int i = 0; i < headers.Length; i++)
            {
                if (headers[i] == selectedEnvironment)
                {
                    envColumnIndex = i;
                    break;
                }
            }

            if (envColumnIndex > -1)
            {
                for (int i = 1; i < envManagerData.Count; i++) // Start from row after headers
                {
                    if (envManagerData[i][0] == "sondbUserpwd") // Matching key
                    {
                        envManagerData[i][envColumnIndex] = dbPassword; // Update password in correct environment column
                        break;
                    }
                }

                File.WriteAllLines(envManagerFile, envManagerData.Select(line => string.Join(",", line)));
                MessageBox.Show("Config and Environment Manager files updated successfully!", "Success", MessageBoxButtons.OK, MessageBoxIcon.Information);
            }
            else
            {
                MessageBox.Show("Environment not found in Environment_Manager.csv!", "Error", MessageBoxButtons.OK, MessageBoxIcon.Error);
            }
        }

        private void Form1_Load(object sender, EventArgs e)
        {
            if (envComboBox.Items.Count > 0) // Ensure dropdown has items
            {
                envComboBox.SelectedIndex = 0; // Set default selection
            }
        }
        private void Form1_Load_1(object sender, EventArgs e)
        {
            
        }

    }
}
